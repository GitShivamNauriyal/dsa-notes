---
tags: [cpp, concurrency, scratch, thread-pool, priority-pool, work-stealing]
links: ["[[08_CPS_Index]]", "[[08_CPS_Bounded_Blocking_Queue]]", "[[08_CPS_Barrier_and_Latch]]"]
---

# Concurrency Primitives From Scratch -- Thread Pools

*<- [[08_CPS_Bounded_Blocking_Queue|Bounded Blocking Queues]] · [[08_CPS_Barrier_and_Latch|Barriers & Latches ->]]*

---

A **Thread Pool** manages a collection of pre-spawned worker threads to process tasks concurrently, mitigating thread construction overhead.

---

## 1. Standard Thread Pool

Features a global thread-safe task queue synchronized using a mutex and condition variable.

```cpp
#include <vector>
#include <thread>
#include <queue>
#include <mutex>
#include <condition_variable>
#include <functional>

class StandardThreadPool {
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    std::mutex mtx;
    std::condition_variable cv;
    bool stop = false;

public:
    explicit StandardThreadPool(std::size_t threads) {
        for (std::size_t i = 0; i < threads; ++i) {
            workers.emplace_back([this]() {
                while (true) {
                    std::function<void()> task;
                    {
                        std::unique_lock<std::mutex> lock(this->mtx);
                        this->cv.wait(lock, [this]() {
                            return this->stop || !this->tasks.empty();
                        });
                        
                        if (this->stop && this->tasks.empty()) {
                            return;
                        }
                        
                        task = std::move(this->tasks.front());
                        this->tasks.pop();
                    }
                    task(); // Execute task
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

    ~StandardThreadPool() {
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

## 2. Priority-Aware Thread Pool

Uses a priority queue to ensure highest-priority submitted tasks are dequeued first.

```cpp
#include <vector>
#include <thread>
#include <queue>
#include <mutex>
#include <condition_variable>
#include <functional>

class PriorityThreadPool {
    struct PrioritizedTask {
        int priority;
        std::function<void()> func;
        
        bool operator<(const PrioritizedTask& other) const {
            return priority > other.priority; // Lower integer value = higher priority
        }
    };

    std::vector<std::thread> workers;
    std::priority_queue<PrioritizedTask> tasks;
    std::mutex mtx;
    std::condition_variable cv;
    bool stop = false;

public:
    explicit PriorityThreadPool(std::size_t threads) {
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
                        
                        task = std::move(this->tasks.top().func);
                        this->tasks.pop();
                    }
                    task();
                }
            });
        }
    }

    void submit(int priority, std::function<void()> task) {
        {
            std::lock_guard<std::mutex> lock(mtx);
            tasks.push({priority, task});
        }
        cv.notify_one();
    }

    ~PriorityThreadPool() {
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

## 3. Work-Stealing Thread Pool (High-Throughput Scheduler)

### The Contention Bottleneck
In a standard thread pool, all threads lock the same global queue. Under high thread count, threads spend significant CPU cycles spinning/waiting on the global queue lock.

### The Work-Stealing Solution
Each worker thread maintains its own **local double-ended queue** (deque) of tasks:
- A worker pushes and pops tasks from the **front** of its own local deque (mostly lock-free).
- If a worker's local deque is empty, it acts as a thief: it picks another worker at random and tries to **steal** a task from the **back** of that worker's deque, drastically reducing global locks and distributing load.

```cpp
#include <vector>
#include <thread>
#include <deque>
#include <mutex>
#include <functional>
#include <atomic>

class WorkStealingPool {
    struct WorkQueue {
        std::deque<std::function<void()>> queue;
        std::mutex mtx;
        
        void push(std::function<void()> task) {
            std::lock_guard<std::mutex> lock(mtx);
            queue.push_front(task);
        }
        
        bool pop(std::function<void()>& task) {
            std::lock_guard<std::mutex> lock(mtx);
            if (queue.empty()) return false;
            task = std::move(queue.front());
            queue.pop_front();
            return true;
        }
        
        bool steal(std::function<void()>& task) {
            std::lock_guard<std::mutex> lock(mtx);
            if (queue.empty()) return false;
            task = std::move(queue.back()); // Steal from the back!
            queue.pop_back();
            return true;
        }
    };

    std::vector<std::thread> workers;
    std::vector<WorkQueue> localQueues;
    std::atomic<bool> stop{false};
    std::size_t numThreads;

public:
    explicit WorkStealingPool(std::size_t threads) : numThreads(threads), localQueues(threads) {
        for (std::size_t i = 0; i < threads; ++i) {
            workers.emplace_back([this, i]() {
                while (!stop) {
                    std::function<void()> task;
                    
                    // 1. Try to pop from local queue
                    if (localQueues[i].pop(task)) {
                        task();
                        continue;
                    }
                    
                    // 2. Local queue empty: try to steal from other queues
                    bool stolen = false;
                    for (std::size_t j = 0; j < numThreads; ++j) {
                        std::size_t targetIdx = (i + j + 1) % numThreads;
                        if (localQueues[targetIdx].steal(task)) {
                            task();
                            stolen = true;
                            break;
                        }
                    }
                    
                    if (!stolen) {
                        std::this_thread::yield(); // Backoff
                    }
                }
            });
        }
    }

    void submitToThread(std::size_t threadId, std::function<void()> task) {
        localQueues[threadId % numThreads].push(task);
    }

    ~WorkStealingPool() {
        stop = true;
        for (auto& worker : workers) {
            if (worker.joinable()) worker.join();
        }
    }
};
```

---

*<- [[08_CPS_Bounded_Blocking_Queue|Bounded Blocking Queues]] · [[08_CPS_Barrier_and_Latch|Barriers & Latches ->]]*
