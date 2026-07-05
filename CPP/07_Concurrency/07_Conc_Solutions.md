---
tags: [cpp, concurrency, solutions]
links: ["[[07_Conc_Index]]", "[[07_Conc_Problems]]", "[[07_Conc_Tricky]]"]
---

# Concurrency in C++ -- Solutions

*<- [[07_Conc_Problems|Problems]] · [[07_Conc_Tricky|Tricky Concurrency Gotchas ->]]*

---

## Tier 1 -- Coordination Basics

### Solution 1.1: Print in Order
We use two C++20 `std::binary_semaphore` objects to control the order of execution between the three threads.

```cpp
#include <semaphore>

class Foo {
    std::binary_semaphore sem1{0}; // blocks second()
    std::binary_semaphore sem2{0}; // blocks third()
public:
    void first(std::function<void()> printFirst) {
        printFirst();
        sem1.release(); // Allow second() to run
    }

    void second(std::function<void()> printSecond) {
        sem1.acquire(); // Wait for first()
        printSecond();
        sem2.release(); // Allow third() to run
    }

    void third(std::function<void()> printThird) {
        sem2.acquire(); // Wait for second()
        printThird();
    }
};
```

---

### Solution 1.2: Print FooBar Alternately
We use two semaphores to alternate execution.

```cpp
#include <semaphore>

class FooBar {
private:
    int n;
    std::binary_semaphore semFoo{1}; // foo starts enabled
    std::binary_semaphore semBar{0}; // bar starts disabled

public:
    FooBar(int n) : n(n) {}

    void foo(std::function<void()> printFoo) {
        for (int i = 0; i < n; i++) {
            semFoo.acquire();
            printFoo();
            semBar.release();
        }
    }

    void bar(std::function<void()> printBar) {
        for (int i = 0; i < n; i++) {
            semBar.acquire();
            printBar();
            semFoo.release();
        }
    }
};
```

---

## Tier 2 -- Advanced State Coordination

### Solution 2.1: Print Zero Even Odd
We coordinate three threads using three semaphores. The `zero` thread determines whether the next number to print is odd or even and releases the appropriate semaphore.

```cpp
#include <semaphore>

class ZeroEvenOdd {
private:
    int n;
    std::binary_semaphore semZero{1};
    std::binary_semaphore semEven{0};
    std::binary_semaphore semOdd{0};

public:
    ZeroEvenOdd(int n) : n(n) {}

    void zero(std::function<void(int)> printNumber) {
        for (int i = 1; i <= n; ++i) {
            semZero.acquire();
            printNumber(0);
            if (i % 2 == 1) {
                semOdd.release();
            } else {
                semEven.release();
            }
        }
    }

    void even(std::function<void(int)> printNumber) {
        for (int i = 2; i <= n; i += 2) {
            semEven.acquire();
            printNumber(i);
            semZero.release();
        }
    }

    void odd(std::function<void(int)> printNumber) {
        for (int i = 1; i <= n; i += 2) {
            semOdd.acquire();
            printNumber(i);
            semZero.release();
        }
    }
};
```

---

### Solution 2.2: Fizz Buzz Multithreaded
We use a mutex and a condition variable to check state conditions on an incrementing counter.

```cpp
#include <mutex>
#include <condition_variable>

class FizzBuzz {
private:
    int n;
    int curr = 1;
    std::mutex mtx;
    std::condition_variable cv;

public:
    FizzBuzz(int n) : n(n) {}

    void fizz(std::function<void()> printFizz) {
        while (true) {
            std::unique_lock<std::mutex> lock(mtx);
            cv.wait(lock, [this]() { return curr > n || (curr % 3 == 0 && curr % 5 != 0); });
            if (curr > n) return;
            printFizz();
            curr++;
            cv.notify_all();
        }
    }

    void buzz(std::function<void()> printBuzz) {
        while (true) {
            std::unique_lock<std::mutex> lock(mtx);
            cv.wait(lock, [this]() { return curr > n || (curr % 5 == 0 && curr % 3 != 0); });
            if (curr > n) return;
            printBuzz();
            curr++;
            cv.notify_all();
        }
    }

    void fizzbuzz(std::function<void()> printFizzBuzz) {
        while (true) {
            std::unique_lock<std::mutex> lock(mtx);
            cv.wait(lock, [this]() { return curr > n || (curr % 3 == 0 && curr % 5 == 0); });
            if (curr > n) return;
            printFizzBuzz();
            curr++;
            cv.notify_all();
        }
    }

    void number(std::function<void(int)> printNumber) {
        while (true) {
            std::unique_lock<std::mutex> lock(mtx);
            cv.wait(lock, [this]() { return curr > n || (curr % 3 != 0 && curr % 5 != 0); });
            if (curr > n) return;
            printNumber(curr);
            curr++;
            cv.notify_all();
        }
    }
};
```

---

## Tier 3 -- Real-World Systems Coordination

### Solution 3.1: Building H2O
We use counting semaphores to coordinate molecule releases. We reset semaphores after releasing a molecule group.

```cpp
#include <semaphore>

class H2O {
    std::counting_semaphore<2> semH{2}; // allow up to 2 H
    std::counting_semaphore<1> semO{1}; // allow up to 1 O
    
    std::binary_semaphore barrierH{0};
    std::binary_semaphore barrierO{0};
    
    std::atomic<int> hCount{0};

public:
    H2O() = default;

    void hydrogen(std::function<void()> releaseHydrogen) {
        semH.acquire();
        hCount++;
        
        if (hCount == 2) {
            // Signal oxygen that H is ready
            barrierO.release();
            // Wait for oxygen to complete the group
            barrierH.acquire();
        }
        
        releaseHydrogen();
        
        if (hCount == 0) {
            // Last hydrogen releases locks for next molecule
            semH.release();
            semH.release();
            semO.release();
        }
    }

    void oxygen(std::function<void()> releaseOxygen) {
        semO.acquire();
        
        // Wait for 2 hydrogens to arrive
        barrierO.acquire();
        
        releaseOxygen();
        
        hCount = 0; // reset
        // Release H threads to complete printing
        barrierH.release();
    }
};
```

---

### Solution 3.2: Dining Philosophers (Deadlock Prevention)
We use `std::scoped_lock` to acquire both forks simultaneously, preventing partial lock deadlocks.

```cpp
#include <mutex>
#include <vector>

class DiningPhilosophers {
    std::vector<std::mutex> forks{5};
public:
    DiningPhilosophers() = default;

    void wantsToEat(int philosopher,
                    std::function<void()> pickLeftFork,
                    std::function<void()> pickRightFork,
                    std::function<void()> eat,
                    std::function<void()> putLeftFork,
                    std::function<void()> putRightFork) {
        
        int left = philosopher;
        int right = (philosopher + 1) % 5;
        
        // Acquire both forks simultaneously using scoped_lock
        {
            std::scoped_lock lock(forks[left], forks[right]);
            pickLeftFork();
            pickRightFork();
            eat();
            putLeftFork();
            putRightFork();
        }
    }
};
```

---

### Solution 3.3: Bounded Blocking Queue

```cpp
#include <mutex>
#include <condition_variable>
#include <queue>

class BoundedBlockingQueue {
    std::queue<int> q;
    std::size_t capacity;
    std::mutex mtx;
    std::condition_variable cv_empty;
    std::condition_variable cv_full;

public:
    BoundedBlockingQueue(int cap) : capacity(cap) {}

    void enqueue(int element) {
        std::unique_lock<std::mutex> lock(mtx);
        cv_full.wait(lock, [this]() { return q.size() < capacity; });
        
        q.push(element);
        cv_empty.notify_one(); // Wake up consumers
    }

    int dequeue() {
        std::unique_lock<std::mutex> lock(mtx);
        cv_empty.wait(lock, [this]() { return !q.empty(); });
        
        int val = q.front();
        q.pop();
        cv_full.notify_one(); // Wake up producers
        return val;
    }
};
```

---

## Tier 4 -- Systems-Hard (Original Challenges)

### Solution 4.1: Custom Priority-Aware Task Scheduler

```cpp
#include <mutex>
#include <condition_variable>
#include <queue>
#include <functional>
#include <vector>

struct Task {
    int priority;
    std::function<void()> func;
    
    // Sort logic: higher priority (lower integer value) goes first
    bool operator<(const Task& other) const {
        return priority > other.priority; 
    }
};

class PriorityScheduler {
    std::priority_queue<Task> tasks;
    std::mutex mtx;
    std::condition_variable cv;

public:
    void submit(int priority, std::function<void()> func) {
        {
            std::lock_guard<std::mutex> lock(mtx);
            tasks.push({priority, func});
        }
        cv.notify_one(); // Wake up one worker thread
    }

    std::function<void()> getTask() {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [this]() { return !tasks.empty(); });
        
        auto task = tasks.top();
        tasks.pop();
        return task.func;
    }
};
```

---

### Solution 4.2: N-Thread Cyclic Execution Chain
We coordinate $N$ threads using $N$ binary semaphores arranged in a ring.

```cpp
#include <vector>
#include <semaphore>
#include <thread>
#include <iostream>

class CyclicChain {
    int N;
    int M;
    std::vector<std::binary_semaphore> sems;

public:
    // Create N semaphores. Thread 0 starts enabled (sem 0 = 1), others disabled (0)
    CyclicChain(int threads_count, int cycles) : N(threads_count), M(cycles) {
        sems.reserve(N);
        sems.emplace_back(1); // Enable first thread
        for (int i = 1; i < N; ++i) {
            sems.emplace_back(0); // Disable others
        }
    }

    void executeThread(int id) {
        for (int cycle = 0; cycle < M; ++cycle) {
            sems[id].acquire(); // Wait for my turn
            
            std::cout << "Thread " << id << " running (Cycle " << cycle << ")\n";
            
            // Release the next thread in the ring
            sems[(id + 1) % N].release(); 
        }
    }
};

void runCyclicChainDemo() {
    int N = 4;
    int M = 3;
    CyclicChain chain(N, M);
    std::vector<std::thread> threads;

    for (int i = 0; i < N; ++i) {
        threads.push_back(std::thread(&CyclicChain::executeThread, &chain, i));
    }
    for (auto& t : threads) t.join();
}
```

---

*<- [[07_Conc_Problems|Problems]] · [[07_Conc_Tricky|Tricky Concurrency Gotchas ->]]*
