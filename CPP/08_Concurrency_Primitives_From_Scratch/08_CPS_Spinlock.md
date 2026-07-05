---
tags: [cpp, concurrency, scratch, spinlock, backoff, ticket-lock]
links: ["[[08_CPS_Index]]", "[[08_CPS_Mutex]]"]
---

# Concurrency Primitives From Scratch -- Spinlocks

*<- [[08_CPS_Index|Index]] · [[08_CPS_Mutex|Mutexes ->]]*

---

A **spinlock** is a low-level synchronization primitive that causes a thread to spin in a tight loop (busy-wait) until the lock becomes available. Spinlocks are ideal when the lock is held for extremely short durations, avoiding the costly context-switch overhead of suspending a thread.

---

## 1. Basic CAS Spinlock

A basic spinlock uses a single atomic boolean flag. It attempts to acquire the lock using Compare-And-Swap (CAS) with `memory_order_acquire`.

```cpp
#include <atomic>

class BasicSpinlock {
    std::atomic<bool> locked{false};
public:
    void lock() {
        // Spin until we successfully swap false to true.
        // We use relaxed load first to avoid polluting the cache bus with writes.
        while (locked.load(std::memory_order_relaxed) || 
               locked.exchange(true, std::memory_order_acquire)) {
            // Busy-wait
        }
    }

    void unlock() {
        locked.store(false, std::memory_order_release);
    }
};
```

---

## 2. Spinlock with Exponential Backoff

### The Cache Coherency Problem
Under high thread contention, if multiple threads spin in a tight loop performing atomic write instructions (like `exchange` or CAS), they flood the CPU cache coherency bus with invalidation messages. This is known as **cache line bouncing** or a **cache storm**, and it severely degrades system-wide CPU throughput.

### The Fix: Exponential Backoff
If a thread fails to acquire the lock, it yields or pauses for an increasing duration before retrying. This dramatically reduces bus traffic.

```cpp
#include <atomic>
#include <thread>
#include <chrono>

#if defined(__x86_64__) || defined(_M_X64)
#include <immintrin.h> // for _mm_pause
#endif

class BackoffSpinlock {
    std::atomic<bool> locked{false};
public:
    void lock() {
        int limit = 32;
        while (locked.load(std::memory_order_relaxed) || 
               locked.exchange(true, std::memory_order_acquire)) {
            
            // Exponential backoff delay
            for (int i = 0; i < limit; ++i) {
                #if defined(__x86_64__) || defined(_M_X64)
                _mm_pause(); // Hint to CPU to optimize spin loops (reduces power/bus traffic)
                #else
                std::this_thread::yield();
                #endif
            }
            if (limit < 1024) limit *= 2; // Double the backoff delay
        }
    }

    void unlock() {
        locked.store(false, std::memory_order_release);
    }
};
```

---

## 3. Ticket Spinlock (Fair FIFO Locking)

### The Unfairness Problem
A standard CAS spinlock is unfair. When the lock is released, any spinning thread can grab it, frequently allowing newly arrived threads to starve older threads that have been waiting.

### The Ticket Lock Solution
Implement a queueing system similar to taking a number ticket at a bakery.
- **`next_ticket`**: Incremented when a thread requests the lock.
- **`now_serving`**: Incremented when a thread releases the lock.
- A thread takes a ticket number and spins until `now_serving` matches its ticket. This guarantees strict FIFO order.

```cpp
#include <atomic>

class TicketSpinlock {
    std::atomic<unsigned int> next_ticket{0};
    std::atomic<unsigned int> now_serving{0};
public:
    void lock() {
        // Atomically claim a ticket
        unsigned int my_ticket = next_ticket.fetch_add(1, std::memory_order_relaxed);
        
        // Spin until our ticket number is called
        while (now_serving.load(std::memory_order_acquire) != my_ticket) {
            #if defined(__x86_64__) || defined(_M_X64)
            _mm_pause();
            #endif
        }
    }

    void unlock() {
        // Call the next ticket in line
        unsigned int curr = now_serving.load(std::memory_order_relaxed);
        now_serving.store(curr + 1, std::memory_order_release);
    }
};
```

---

*<- [[08_CPS_Index|Index]] · [[08_CPS_Mutex|Mutexes ->]]*
