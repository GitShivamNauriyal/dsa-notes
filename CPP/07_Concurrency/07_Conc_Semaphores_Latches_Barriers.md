---
tags: [cpp, concurrency, semaphore, latch, barrier, Cpp20]
links: ["[[07_Conc_Index]]", "[[07_Conc_Atomics_and_Memory_Model]]", "[[07_Conc_Lock_Free_Basics]]"]
---

# Concurrency in C++ -- Semaphores, Latches, & Barriers

*<- [[07_Conc_Atomics_and_Memory_Model|Atomics & Memory Model]] · [[07_Conc_Lock_Free_Basics|Lock-Free Basics ->]]*

---

C++20 introduced high-performance coordination primitives that bypass the resource overhead of combining mutexes and condition variables.

---

## 1. Semaphores (`std::counting_semaphore`, `std::binary_semaphore`)

A **semaphore** is a lightweight, counter-based signaling primitive.
- **`acquire()`**: Decrements the counter. If the counter is 0, the thread blocks until it is incremented.
- **`release(pt = 1)`**: Increments the counter and wakes up waiting threads.
- **`std::binary_semaphore`**: Shorthand for `std::counting_semaphore<1>`, which can be used as a fast, light-weight lock.

### Thread Alternation Example (Ping-Pong)
```cpp
#include <semaphore>
#include <thread>
#include <iostream>

std::binary_semaphore semA{1}; // Start with A enabled
std::binary_semaphore semB{0}; // Start with B disabled

void workerA() {
    for (int i = 0; i < 3; ++i) {
        semA.acquire();
        std::cout << "Ping ";
        semB.release();
    }
}

void workerB() {
    for (int i = 0; i < 3; ++i) {
        semB.acquire();
        std::cout << "Pong\n";
        semA.release();
    }
}

void demoSemaphore() {
    std::thread t1(workerA);
    std::thread t2(workerB);
    t1.join();
    t2.join();
}
```

---

## 2. Latches (`std::latch`)

A **latch** is a single-use down-counter that blocks threads until the counter hits 0.
- Once the counter reaches 0, the latch remains open forever; subsequent calls to `wait()` return immediately.
- **`arrive_and_wait()`**: Decrements the latch count by 1 and blocks until the count reaches 0.

### Example: Coordinating Thread Startup
```cpp
#include <latch>
#include <thread>
#include <vector>
#include <iostream>

void initializeService(std::latch& startupLatch, int id) {
    std::cout << "Service " << id << " initialized.\n";
    // Decrement the latch and wait for all other threads to finish initialization
    startupLatch.arrive_and_wait(); 
    
    std::cout << "Service " << id << " started running!\n";
}

void demoLatch() {
    // 3 services + 1 main thread = 4
    std::latch startupLatch(3); 
    std::vector<std::thread> threads;

    for (int i = 1; i <= 3; ++i) {
        threads.push_back(std::thread(initializeService, std::ref(startupLatch), i));
    }

    for (auto& t : threads) t.join();
}
```

---

## 3. Barriers (`std::barrier`)

Unlike `std::latch`, a **barrier** is a **reusable** synchronization point.
- A fixed set of threads block at the barrier until all have arrived.
- Once the last thread arrives, the barrier runs a user-defined **completion function**, resets its internal counter, and releases all threads to begin the next phase.

```cpp
#include <barrier>
#include <thread>
#include <vector>
#include <iostream>

// Reusable completion function
auto onCompletion = []() noexcept {
    std::cout << "--- Phase Completed ---\n";
};

// 3 threads coordinating over 2 phases
std::barrier syncPoint(3, onCompletion);

void runPhaseTask(int id) {
    std::cout << "Thread " << id << ": Executing Phase 1\n";
    syncPoint.arrive_and_wait(); // Block until all 3 threads finish Phase 1
    
    std::cout << "Thread " << id << ": Executing Phase 2\n";
    syncPoint.arrive_and_wait(); // Block until all 3 threads finish Phase 2
}

void demoBarrier() {
    std::vector<std::thread> threads;
    for (int i = 1; i <= 3; ++i) {
        threads.push_back(std::thread(runPhaseTask, i));
    }
    for (auto& t : threads) t.join();
}
```

---

*<- [[07_Conc_Lock_Free_Basics|Lock-Free Basics]] · [[07_Conc_Lock_Free_Basics|Lock-Free Basics ->]]*
