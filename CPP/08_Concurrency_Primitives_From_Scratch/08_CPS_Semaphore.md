---
tags: [cpp, concurrency, scratch, semaphore, binary-semaphore, counting-semaphore, atomics-wait]
links: ["[[08_CPS_Index]]", "[[08_CPS_Mutex]]", "[[08_CPS_Read_Write_Lock]]"]
---

# Concurrency Primitives From Scratch -- Semaphores

*<- [[08_CPS_Mutex|Mutexes]] · [[08_CPS_Read_Write_Lock|Read-Write Locks ->]]*

---

A **semaphore** coordinates execution flow using an internal counter.

---

## 1. Custom Binary Semaphore (Mutex + Condvar)

Exposes a binary state counter restricted to $[0, 1]$.

```cpp
#include <mutex>
#include <condition_variable>

class CustomBinarySemaphore {
    std::mutex mtx;
    std::condition_variable cv;
    bool signaled = false;

public:
    explicit CustomBinarySemaphore(bool initialSignaled = false) : signaled(initialSignaled) {}

    void acquire() {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [this]() { return signaled; });
        signaled = false; // Reset signal
    }

    void release() {
        std::unique_lock<std::mutex> lock(mtx);
        signaled = true;
        cv.notify_one();
    }
};
```

---

## 2. Custom Counting Semaphore (Mutex + Condvar)

Manages a counter up to $N$ to regulate access to a pool of resources.

```cpp
#include <mutex>
#include <condition_variable>

class CustomCountingSemaphore {
    std::mutex mtx;
    std::condition_variable cv;
    int count;

public:
    explicit CustomCountingSemaphore(int initialCount) : count(initialCount) {}

    void acquire() {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [this]() { return count > 0; });
        count--;
    }

    void release(int releaseCount = 1) {
        std::unique_lock<std::mutex> lock(mtx);
        count += releaseCount;
        if (releaseCount > 1) {
            cv.notify_all();
        } else {
            cv.notify_one();
        }
    }
};
```

---

## 3. Atomics-Only Counting Semaphore (C++20 `atomic::wait` / Futex Style)

C++20 introduced **`atomic::wait()`** and **`atomic::notify_one()/notify_all()`**. This allows a thread to sleep on an atomic variable at the OS kernel level (equivalent to Linux **`futex`**), bypassing condition variables.

```cpp
#include <atomic>

class AtomicsCountingSemaphore {
    std::atomic<int> count;

public:
    explicit AtomicsCountingSemaphore(int initialCount) : count(initialCount) {}

    void acquire() {
        while (true) {
            int current = count.load(std::memory_order_relaxed);
            
            // If count is greater than 0, try to decrement it
            if (current > 0) {
                if (count.compare_exchange_weak(current, current - 1, 
                                                std::memory_order_acquire, 
                                                std::memory_order_relaxed)) {
                    return; // Successfully acquired!
                }
                continue; // CAS failed (race condition), retry immediately
            }
            
            // If count is 0, block the thread.
            // C++20 atomic::wait blocks until the atomic variable changes value.
            count.wait(0, std::memory_order_relaxed); 
        }
    }

    void release(int releaseCount = 1) {
        // Increment the count atomically
        count.fetch_add(releaseCount, std::memory_order_release);
        
        // C++20 notify wakes up waiting threads blocked in atomic::wait
        if (releaseCount > 1) {
            count.notify_all();
        } else {
            count.notify_one();
        }
    }
};
```

---

*<- [[08_CPS_Mutex|Mutexes]] · [[08_CPS_Read_Write_Lock|Read-Write Locks ->]]*
