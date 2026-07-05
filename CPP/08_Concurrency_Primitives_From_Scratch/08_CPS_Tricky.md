---
tags: [cpp, concurrency, scratch, tricky, race-conditions, destructor-cleanup]
links: ["[[08_CPS_Index]]", "[[08_CPS_Solutions]]", "[[../09_Performance_and_Optimization/09_Perf_Index]]"]
---

# Concurrency Primitives From Scratch -- Tricky & Higher-Order

*<- [[08_CPS_Solutions|Solutions]] · [[../09_Performance_and_Optimization/09_Perf_Index|Performance & Optimization ->]]*

---

Implementing low-level primitives from scratch exposes systems programmers to several subtle and destructive race conditions.

---

## 1. Naive Resets in Barriers (Generation Desynchronization)

A naive barrier decreases a counter on arrival. When the counter reaches 0, the last thread resets the counter to $N$ and calls `notify_all()`.

### The Race Condition
A fast thread awakened by `notify_all()` can exit the barrier, loop through its worker code, and call `arrive_and_wait()` again *before* a slower thread has even exited the first condition variable sleep.
- The fast thread will decrement the counter (which was reset to $N$).
- When the slow thread finally wakes up and tries to clean up or proceed, the state counter is corrupted. This frequently leads to deadlocks or early phase triggers.
- **The Fix**: Incorporate a **generation phase counter**. Threads check the generation ID, and wait until it increments.

---

## 2. Spurious Wakeups on Condition Variables

When wrapping custom locks using `std::condition_variable`, you cannot rely on a simple `cv.wait(lock)` call. A thread can wake up spuriously due to OS kernel interrupts.
- If you do not supply a state checking loop predicate, the thread will proceed to execute as if it acquired the lock, triggering severe data races.
- **Rule**: Always verify state flags inside a loop predicate: `cv.wait(lock, [this] { return state_is_ready; });`.

---

## 3. Destructor Cleanup Race Conditions (Dangling Pointers)

When writing thread-safe queues or thread pools, a common error is deleting resources while worker threads are suspended inside CV wait calls.

```cpp
class CustomQueue {
    std::mutex mtx;
    std::condition_variable cv;
    // ...
public:
    ~CustomQueue() {
        // BUG: If workers are suspended inside cv.wait(), deleting the queue
        // deletes the mutex and cv while threads are waiting on them!
        // This triggers Undefined Behavior (usually kernel crashes).
    }
};
```

### The Safe Destruction Process:
1. Signal a state flag (e.g. `stop = true`).
2. Lock the mutex and wake up all waiting threads via `cv.notify_all()`.
3. Join all worker threads *before* releasing the queue's internal storage resources.

---

## Recap of Tricky Custom Primitive Gotchas

| Gotcha | Theme |
|---|---|
| **Generation Desynchronization** | Fast threads can loop back and decrement a barrier's count early; resolved by advancing generation IDs. |
| **Spurious CV Wakeups** | Threads can wake up without notice; state checks must always run inside a predicate loop. |
| **Destructor Cleanup** | Deleting synchronization primitives while threads are blocked on them causes UB; join threads before freeing. |

---

*<- [[08_CPS_Solutions|Solutions]] · [[../09_Performance_and_Optimization/09_Perf_Index|Performance & Optimization ->]]*
