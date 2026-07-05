---
tags: [cpp, concurrency, mutex, lock-managers, condition-variable, deadlocks, spurious-wakeups]
links: ["[[07_Conc_Index]]", "[[07_Conc_Threading_Basics]]", "[[07_Conc_Atomics_and_Memory_Model]]"]
---

# Concurrency in C++ -- Synchronization Primitives

*<- [[07_Conc_Threading_Basics|Threading Basics]] · [[07_Conc_Atomics_and_Memory_Model|Atomics & Memory Model ->]]*

---

## 1. The Mutex and Lock-Manager Families

To protect shared resources from data races, C++ offers several mutex classes and RAII-based lock managers.

### Mutex Variants
- **`std::mutex`**: Basic mutual exclusion lock. Cannot be locked recursively by the same thread.
- **`std::recursive_mutex`**: Allows the owning thread to acquire the lock multiple times recursively (requires matching unlocks). Avoid where possible due to higher overhead and design smell.
- **`std::shared_mutex`** *(C++14)*: Read-Write lock. Allows multiple threads to share read access, but only one thread to write.

---

### Lock Managers (RAII Wrappers)

| Wrapper | Overhead | Movable? | Key Use Case |
|---|---|---|---|
| **`std::lock_guard`** | Low (zero cost) | No | Simple scope-bound locking |
| **`std::unique_lock`** | Medium | **Yes** | Lazy locking, unlocking, and `condition_variable` waits |
| **`std::scoped_lock`** *(C++17)*| Low-Medium | No | Safely locking multiple mutexes simultaneously |

```cpp
#include <mutex>
#include <shared_mutex>

std::shared_mutex rwMutex;
int sharedData = 0;

// Multiple threads can execute readData simultaneously
int readData() {
    std::shared_lock<std::shared_mutex> lock(rwMutex); // Shared lock (Read)
    return sharedData;
}

// Only one thread can execute writeData at a time
void writeData(int val) {
    std::unique_lock<std::shared_mutex> lock(rwMutex); // Exclusive lock (Write)
    sharedData = val;
}
```

---

## 2. Deadlock Avoidance and `std::scoped_lock` (C++17)

A **deadlock** occurs when Thread 1 holds Lock A and waits for Lock B, while Thread 2 holds Lock B and waits for Lock A. Neither thread can proceed.

### Avoidance Strategies:
1. **Strict Lock Ordering**: Always acquire locks in the same order throughout the codebase (e.g. Lock A first, then Lock B).
2. **`std::scoped_lock`**: C++17 RAII manager that accepts multiple mutexes and locks them simultaneously using a deadlock-avoidance algorithm (similar to `std::lock`).

```cpp
#include <mutex>

struct Account {
    std::mutex mtx;
    int balance;
};

void transfer(Account& from, Account& to, int amount) {
    // Lock both mutexes safely without risking deadlocks
    std::scoped_lock lock(from.mtx, to.mtx); 
    
    from.balance -= amount;
    to.balance += amount;
}
```

---

## 3. Condition Variables & Spurious Wakeups

`std::condition_variable` allows a thread to suspend execution until it is signaled by another thread that a condition is met.

> [!IMPORTANT]
> **Spurious Wakeups**: A waiting thread can wake up even if no one signaled the condition variable.
> **The Fix**: Always wrap the wait statement in a **while-loop** or pass a **predicate lambda** to `wait()`. This ensures the thread checks the condition again upon waking and suspends if the signal was spurious.

```cpp
#include <mutex>
#include <condition_variable>
#include <queue>
#include <iostream>

class ThreadSafeQueue {
    std::queue<int> q;
    std::mutex mtx;
    std::condition_variable cv;

public:
    void push(int val) {
        {
            std::lock_guard<std::mutex> lock(mtx);
            q.push(val);
        }
        cv.notify_one(); // Wake up one waiting thread
    }

    int pop() {
        std::unique_lock<std::mutex> lock(mtx);
        
        // Pass a predicate to wait() to protect against spurious wakeups!
        // wait() unlocks the mutex, suspends, and re-locks the mutex when awakened.
        cv.wait(lock, [this]() { return !q.empty(); });
        
        int val = q.front();
        q.pop();
        return val;
    }
};
```

---

*<- [[07_Conc_Threading_Basics|Threading Basics]] · [[07_Conc_Atomics_and_Memory_Model|Atomics & Memory Model ->]]*
