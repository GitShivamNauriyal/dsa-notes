---
tags: [cpp, concurrency, scratch, mutex, recursive-mutex, shared-mutex]
links: ["[[08_CPS_Index]]", "[[08_CPS_Spinlock]]", "[[08_CPS_Semaphore]]"]
---

# Concurrency Primitives From Scratch -- Mutexes

*<- [[08_CPS_Spinlock|Spinlocks]] · [[08_CPS_Semaphore|Semaphores ->]]*

---

Unlike a spinlock, a **mutex** (mutual exclusion) primitive suspends the execution of waiting threads, allowing other OS processes to use the CPU.

---

## 1. Custom Mutex

This implementation uses an atomic state flag for fast-path locking, falling back to a condition variable to sleep/wake threads.

```cpp
#include <mutex>
#include <condition_variable>
#include <atomic>

class CustomMutex {
    std::atomic<bool> locked{false};
    std::mutex internalMtx;
    std::condition_variable cv;
public:
    void lock() {
        // Fast path: if unlocked, acquire immediately
        if (!locked.exchange(true, std::memory_order_acquire)) {
            return;
        }

        // Slow path: suspend thread until notified
        std::unique_lock<std::mutex> lock(internalMtx);
        cv.wait(lock, [this]() {
            // exchange returns old state. If false, we successfully acquired the lock!
            return !locked.exchange(true, std::memory_order_acquire);
        });
    }

    void unlock() {
        locked.store(false, std::memory_order_release);
        cv.notify_one(); // Wake up one waiting thread
    }
};
```

---

## 2. Custom Recursive Mutex

A recursive mutex allows the owning thread to acquire the lock multiple times without deadlock. It tracks the owning thread's ID and increments a counter.

```cpp
#include <thread>
#include <atomic>
#include <mutex>
#include <condition_variable>

class CustomRecursiveMutex {
    std::atomic<std::thread::id> ownerId{};
    std::size_t lockCount = 0;
    std::mutex internalMtx;
    std::condition_variable cv;

public:
    void lock() {
        auto self = std::this_thread::get_id();
        
        // If the calling thread already owns the lock, simply increment count
        if (ownerId.load(std::memory_order_relaxed) == self) {
            lockCount++;
            return;
        }

        // Otherwise, acquire exclusive lock ownership
        std::unique_lock<std::mutex> lock(internalMtx);
        cv.wait(lock, [this]() {
            std::thread::id empty{};
            return ownerId.load(std::memory_order_relaxed) == empty;
        });

        ownerId.store(self, std::memory_order_relaxed);
        lockCount = 1;
    }

    void unlock() {
        auto self = std::this_thread::get_id();
        if (ownerId.load(std::memory_order_relaxed) != self) {
            throw std::runtime_error("Attempted to unlock recursive mutex not owned by thread!");
        }

        lockCount--;
        if (lockCount == 0) {
            std::thread::id empty{};
            ownerId.store(empty, std::memory_order_relaxed);
            cv.notify_one(); // Release waiting threads
        }
    }
};
```

---

## 3. Custom Shared (Read-Write) Mutex

Exposes shared lock options for multiple readers, and exclusive lock options for writers.

```cpp
#include <mutex>
#include <condition_variable>

class CustomSharedMutex {
    std::mutex mtx;
    std::condition_variable cv;
    int readers = 0;
    bool writing = false;

public:
    // Exclusive Lock (Writers)
    void lock() {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [this]() { return readers == 0 && !writing; });
        writing = true;
    }

    void unlock() {
        std::unique_lock<std::mutex> lock(mtx);
        writing = false;
        cv.notify_all(); // Wake up waiting readers or writers
    }

    // Shared Lock (Readers)
    void lock_shared() {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [this]() { return !writing; });
        readers++;
    }

    void unlock_shared() {
        std::unique_lock<std::mutex> lock(mtx);
        readers--;
        if (readers == 0) {
            cv.notify_all(); // Wake up blocked writers
        }
    }
};
```

---

*<- [[08_CPS_Spinlock|Spinlocks]] · [[08_CPS_Semaphore|Semaphores ->]]*
