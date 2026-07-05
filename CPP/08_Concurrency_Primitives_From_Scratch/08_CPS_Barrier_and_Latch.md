---
tags: [cpp, concurrency, scratch, barrier, latch, generation-barrier]
links: ["[[08_CPS_Index]]", "[[08_CPS_Thread_Pool]]", "[[08_CPS_Problems]]"]
---

# Concurrency Primitives From Scratch -- Barriers & Latches

*<- [[08_CPS_Thread_Pool|Thread Pools]] · [[08_CPS_Problems|Problems ->]]*

---

A **latch** is a single-use counter-based blocking barrier, whereas a **barrier** is reusable and tracks execution generation phases.

---

## 1. Custom Latch

Counts down to 0, at which point all waiting threads are permanently released.

```cpp
#include <mutex>
#include <condition_variable>

class CustomLatch {
    std::mutex mtx;
    std::condition_variable cv;
    int count;

public:
    explicit CustomLatch(int initialCount) : count(initialCount) {}

    void count_down() {
        std::unique_lock<std::mutex> lock(mtx);
        if (count > 0) {
            count--;
            if (count == 0) {
                cv.notify_all(); // Release all waiting threads
            }
        }
    }

    void wait() {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [this]() { return count == 0; });
    }

    void arrive_and_wait() {
        std::unique_lock<std::mutex> lock(mtx);
        if (count > 0) {
            count--;
            if (count == 0) {
                cv.notify_all();
                return;
            }
        }
        cv.wait(lock, [this]() { return count == 0; });
    }
};
```

---

## 2. Custom Barrier (with Generations)

### The Generation Desynchronization Bug
A naive barrier implementation decrements a counter when threads arrive and resets the counter to `N` when it hits 0.
- **The Bug**: If a fast thread executes its code, loops back to the barrier, and calls `wait()` again *before* slow threads have fully exited the first barrier call, it can decrement the count early. This locks slow threads inside the barrier forever.
- **The Fix**: Track a **generation index** (or phase). Threads suspend until the generation changes (which only happens when the last thread arrives).

```cpp
#include <mutex>
#include <condition_variable>
#include <functional>

class CustomBarrier {
    std::mutex mtx;
    std::condition_variable cv;
    std::size_t numThreads;
    std::size_t waitingThreads = 0;
    std::size_t generation = 0; // Generation tracker
    std::function<void()> completionFunc;

public:
    explicit CustomBarrier(std::size_t threads, std::function<void()> completion = nullptr) 
        : numThreads(threads), completionFunc(completion) {}

    void arrive_and_wait() {
        std::unique_lock<std::mutex> lock(mtx);
        std::size_t currentGeneration = generation;

        waitingThreads++;
        
        // If we are the last thread to arrive
        if (waitingThreads == numThreads) {
            if (completionFunc) {
                completionFunc(); // Run phase completion routine
            }
            
            waitingThreads = 0;   // Reset counter for next phase
            generation++;         // Advance to next generation phase
            cv.notify_all();      // Release all waiting threads of the current generation
        } else {
            // Wait until the generation index has been advanced by the last thread
            cv.wait(lock, [this, currentGeneration]() {
                return generation != currentGeneration;
            });
        }
    }
};
```

---

*<- [[08_CPS_Thread_Pool|Thread Pools]] · [[08_CPS_Problems|Problems ->]]*
