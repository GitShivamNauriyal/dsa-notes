---
tags: [cpp, concurrency, scratch, problems]
links: ["[[08_CPS_Index]]", "[[08_CPS_Barrier_and_Latch]]", "[[08_CPS_Solutions]]"]
---

# Concurrency Primitives From Scratch -- Problems

*<- [[08_CPS_Barrier_and_Latch|Barriers & Latches]] · [[08_CPS_Solutions|Solutions ->]]*

---

Implement the following custom concurrency primitives in C++. Write clean, safe, and compile-ready code for each challenge.

---

### Problem 1: Custom Spinlock with Backoff
Write a class `CustomSpinlock` that provides `lock()` and `unlock()` capabilities using `std::atomic<bool>`.
- The `lock()` method must perform an exponential backoff loop using `_mm_pause()` or `yield()` to prevent cache storming under high contention.

---

### Problem 2: Custom Recursive Mutex
Write a class `CustomRecursiveMutex` that:
- Allows the owning thread to acquire the lock multiple times recursively without self-deadlocking.
- Blocks other threads attempting to acquire the lock.
- Tracks lock count and releases only when the acquisition count drops back to 0.

---

### Problem 3: Custom Counting Semaphore
Write a class `CustomCountingSemaphore` that coordinates execution flow using an internal counter.
- Maintain a count limit $N$.
- `acquire()` blocks if count is 0, otherwise decrements.
- `release(int n)` increments the count and wakes up $n$ waiting threads.
- Implement both a mutex-based version and an atomics-only version.

---

### Problem 4: Custom Read-Write Lock (Writer-Preferred)
Write a class `WriterPreferredRWLock` that:
- Allows multiple concurrent readers.
- Allows exclusive access for a single writer.
- **Prevents writer starvation**: Arriving readers must wait if a writer is active OR if another writer is waiting in line.

---

### Problem 5: Custom Bounded Blocking Queue
Write a class `BoundedBlockingQueue` that holds elements of any generic type `T`.
- The queue must have a fixed capacity.
- `push(const T& element)` blocks if the queue is full.
- `pop()` blocks if the queue is empty.
- Implement it using counting and binary semaphores (do not use condition variables).

---

### Problem 6: Custom Thread Pool
Write a class `CustomThreadPool` that:
- Spawns $K$ worker threads on instantiation.
- Exposes `submit(std::function<void()> task)` to accept jobs.
- Workers block if no tasks are available, executing them in FIFO order.
- Cleanly joins and stops all worker threads when the pool goes out of scope (destructor).

---

### Problem 7: Custom Generation Barrier
Write a class `CustomBarrier` that:
- Accepts a thread count limit $N$ and a completion function on constructor initialization.
- Exposes `arrive_and_wait()`.
- Uses a **generation index** to prevent fast threads from executing early and corrupting subsequent lock phases.

---

*<- [[08_CPS_Barrier_and_Latch|Barriers & Latches]] · [[08_CPS_Solutions|Solutions ->]]*
