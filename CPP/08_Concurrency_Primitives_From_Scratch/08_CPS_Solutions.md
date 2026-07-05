---
tags: [cpp, concurrency, scratch, solutions]
links: ["[[08_CPS_Index]]", "[[08_CPS_Problems]]", "[[08_CPS_Tricky]]"]
---

# Concurrency Primitives From Scratch -- Solutions

*<- [[08_CPS_Problems|Problems]] · [[08_CPS_Tricky|Tricky Scratch Gotchas ->]]*

---

Below are complete, compilable implementations for all the custom concurrency primitives.

---

### Solution 1: Custom Spinlock with Backoff

```cpp
#include <atomic>
#include <thread>

#if defined(__x86_64__) || defined(_M_X64)
#include <immintrin.h>
#endif

class CustomSpinlock {
    std::atomic<bool> locked{false};
public:
    void lock() {
        int limit = 32;
        // Check locked flag first (relaxed) to avoid dirtying the cache line on other cores
        while (locked.load(std::memory_order_relaxed) || 
               locked.exchange(true, std::memory_order_acquire)) {
            
            // Exponential backoff
            for (int i = 0; i < limit; ++i) {
                #if defined(__x86_64__) || defined(_M_X64)
                _mm_pause(); // reduce bus traffic and power usage
                #else
                std::this_thread::yield();
                #endif
            }
            if (limit < 1024) limit *= 2;
        }
    }

    void unlock() {
        locked.store(false, std::memory_order_release);
    }
};
```

---

### Solution 2: Custom Recursive Mutex

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
        
        if (ownerId.load(std::memory_order_relaxed) == self) {
            lockCount++;
            return;
        }

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
            throw std::runtime_error("Cannot unlock mutex: not owned by this thread!");
        }

        lockCount--;
        if (lockCount == 0) {
            std::thread::id empty{};
            ownerId.store(empty, std::memory_order_relaxed);
            cv.notify_one();
        }
    }
};
```

---

### Solution 3: Custom Counting Semaphore

#### Version A: Mutex + Condvar

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

    void release(int n = 1) {
        std::unique_lock<std::mutex> lock(mtx);
        count += n;
        if (n > 1) cv.notify_all();
        else cv.notify_one();
    }
};
```

#### Version B: Atomics-Only (C++20 `atomic::wait` style)

```cpp
#include <atomic>

class AtomicsCountingSemaphore {
    std::atomic<int> count;

public:
    explicit AtomicsCountingSemaphore(int initialCount) : count(initialCount) {}

    void acquire() {
        while (true) {
            int current = count.load(std::memory_order_relaxed);
            if (current > 0) {
                if (count.compare_exchange_weak(current, current - 1, 
                                                std::memory_order_acquire, 
                                                std::memory_order_relaxed)) {
                    return;
                }
                continue;
            }
            count.wait(0, std::memory_order_relaxed); // Block on kernel event (futex)
        }
    }

    void release(int n = 1) {
        count.fetch_add(n, std::memory_order_release);
        if (n > 1) count.notify_all();
        else count.notify_one();
    }
};
```

---

### Solution 4: Custom Read-Write Lock (Writer-Preferred)

```cpp
#include <mutex>
#include <condition_variable>

class WriterPreferredRWLock {
    std::mutex mtx;
    std::condition_variable cv;
    int activeReaders = 0;
    int waitingWriters = 0;
    bool writerActive = false;

public:
    void lockRead() {
        std::unique_lock<std::mutex> lock(mtx);
        // Arriving readers wait if a writer is active OR if any writer is waiting
        cv.wait(lock, [this]() { return !writerActive && waitingWriters == 0; });
        activeReaders++;
    }

    void unlockRead() {
        std::unique_lock<std::mutex> lock(mtx);
        activeReaders--;
        if (activeReaders == 0) {
            cv.notify_all(); // Wake up waiting writers
        }
    }

    void lockWrite() {
        std::unique_lock<std::mutex> lock(mtx);
        waitingWriters++;
        cv.wait(lock, [this]() { return activeReaders == 0 && !writerActive; });
        waitingWriters--;
        writerActive = true;
    }

    void unlockWrite() {
        std::unique_lock<std::mutex> lock(mtx);
        writerActive = false;
        cv.notify_all(); // Wake up queued readers/writers
    }
};
```

---

### Solution 5: Custom Bounded Blocking Queue (Semaphore-Based)

```cpp
#include <semaphore>
#include <vector>

template <typename T>
class BoundedBlockingQueue {
    std::vector<T> buffer;
    std::size_t head = 0;
    std::size_t tail = 0;
    std::size_t capacity;

    std::counting_semaphore<> emptySlots;
    std::counting_semaphore<> filledItems;
    std::binary_semaphore bufferMtx{1};

public:
    explicit BoundedBlockingQueue(std::size_t cap) 
        : buffer(cap), capacity(cap), emptySlots(cap), filledItems(0) {}

    void push(const T& element) {
        emptySlots.acquire(); // Blocks if no empty slots available
        
        bufferMtx.acquire();
        buffer[tail] = element;
        tail = (tail + 1) % capacity;
        bufferMtx.release();
        
        filledItems.release(); // Increment items, wake up consumers
    }

    T pop() {
        filledItems.acquire(); // Blocks if no filled items available
        
        bufferMtx.acquire();
        T item = buffer[head];
        head = (head + 1) % capacity;
        bufferMtx.release();
        
        emptySlots.release(); // Increment slots, wake up producers
        return item;
    }
};
```

---

### Solution 6: Custom Thread Pool

```cpp
#include <vector>
#include <thread>
#include <queue>
#include <mutex>
#include <condition_variable>
#include <functional>

class CustomThreadPool {
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    std::mutex mtx;
    std::condition_variable cv;
    bool stop = false;

public:
    explicit CustomThreadPool(std::size_t threads) {
        for (std::size_t i = 0; i < threads; ++i) {
            workers.emplace_back([this]() {
                while (true) {
                    std::function<void()> task;
                    {
                        std::unique_lock<std::mutex> lock(this->mtx);
                        this->cv.wait(lock, [this]() {
                            return this->stop || !this->tasks.empty();
                        });
                        
                        if (this->stop && this->tasks.empty()) return;
                        
                        task = std::move(this->tasks.front());
                        this->tasks.pop();
                    }
                    task();
                }
            });
        }
    }

    void submit(std::function<void()> task) {
        {
            std::lock_guard<std::mutex> lock(mtx);
            tasks.push(task);
        }
        cv.notify_one();
    }

    ~CustomThreadPool() {
        {
            std::lock_guard<std::mutex> lock(mtx);
            stop = true;
        }
        cv.notify_all();
        for (auto& worker : workers) {
            if (worker.joinable()) worker.join();
        }
    }
};
```

---

### Solution 7: Custom Generation Barrier

```cpp
#include <mutex>
#include <condition_variable>
#include <functional>

class CustomBarrier {
    std::mutex mtx;
    std::condition_variable cv;
    std::size_t numThreads;
    std::size_t waitingThreads = 0;
    std::size_t generation = 0;
    std::function<void()> completionFunc;

public:
    explicit CustomBarrier(std::size_t threads, std::function<void()> completion = nullptr) 
        : numThreads(threads), completionFunc(completion) {}

    void arrive_and_wait() {
        std::unique_lock<std::mutex> lock(mtx);
        std::size_t currentGeneration = generation;

        waitingThreads++;

        if (waitingThreads == numThreads) {
            if (completionFunc) {
                completionFunc();
            }
            waitingThreads = 0;
            generation++;
            cv.notify_all(); // Wake up threads of the current generation
        } else {
            // Loop protects against spurious wakeups, staying inside current generation
            cv.wait(lock, [this, currentGeneration]() {
                return generation != currentGeneration;
            });
        }
    }
};
```

---

*<- [[08_CPS_Problems|Problems]] · [[08_CPS_Tricky|Tricky Scratch Gotchas ->]]*
