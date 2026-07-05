---
tags: [cpp, concurrency, scratch, blocking-queue, semaphore-queue, lock-free-ring-buffer]
links: ["[[08_CPS_Index]]", "[[08_CPS_Read_Write_Lock]]", "[[08_CPS_Thread_Pool]]"]
---

# Concurrency Primitives From Scratch -- Bounded Blocking Queues

*<- [[08_CPS_Read_Write_Lock|Read-Write Locks]] · [[08_CPS_Thread_Pool|Thread Pools ->]]*

---

A **Bounded Blocking Queue** coordinates safe data transfer between producers and consumers, blocking threads when capacity thresholds are breached.

---

## 1. Condition Variable Bounded Queue

Uses two separate condition variables to signal producers and consumers independently, avoiding thundering herd wakeups.

```cpp
#include <mutex>
#include <condition_variable>
#include <vector>

template <typename T>
class CVBoundedQueue {
    std::vector<T> buffer;
    std::size_t head = 0;
    std::size_t tail = 0;
    std::size_t sz = 0;
    std::size_t capacity;
    
    std::mutex mtx;
    std::condition_variable cvEmpty;
    std::condition_variable cvFull;

public:
    explicit CVBoundedQueue(std::size_t cap) : buffer(cap), capacity(cap) {}

    void push(const T& item) {
        std::unique_lock<std::mutex> lock(mtx);
        cvFull.wait(lock, [this]() { return sz < capacity; });

        buffer[tail] = item;
        tail = (tail + 1) % capacity;
        sz++;

        cvEmpty.notify_one(); // Wake up one waiting consumer
    }

    T pop() {
        std::unique_lock<std::mutex> lock(mtx);
        cvEmpty.wait(lock, [this]() { return sz > 0; });

        T item = buffer[head];
        head = (head + 1) % capacity;
        sz--;

        cvFull.notify_one(); // Wake up one waiting producer
        return item;
    }
};
```

---

## 2. Semaphore-Based Bounded Queue

Regulates boundaries using counting semaphores. Extremely elegant and popular in operating systems design.
- **`emptySlots`**: Counting semaphore initialized to `capacity`.
- **`filledItems`**: Counting semaphore initialized to `0`.
- **`mtx`**: Binary semaphore for protecting buffer modifications.

```cpp
#include <semaphore>
#include <vector>

template <typename T>
class SemaphoreBoundedQueue {
    std::vector<T> buffer;
    std::size_t head = 0;
    std::size_t tail = 0;
    std::size_t capacity;

    std::counting_semaphore<> emptySlots;
    std::counting_semaphore<> filledItems;
    std::binary_semaphore bufferMtx{1};

public:
    explicit SemaphoreBoundedQueue(std::size_t cap) 
        : buffer(cap), capacity(cap), emptySlots(cap), filledItems(0) {}

    void push(const T& item) {
        emptySlots.acquire(); // Decrement empty slots; block if 0
        
        bufferMtx.acquire(); // Protect buffer modification
        buffer[tail] = item;
        tail = (tail + 1) % capacity;
        bufferMtx.release();
        
        filledItems.release(); // Increment filled items; wake up consumer
    }

    T pop() {
        filledItems.acquire(); // Decrement filled items; block if 0
        
        bufferMtx.acquire();
        T item = buffer[head];
        head = (head + 1) % capacity;
        bufferMtx.release();
        
        emptySlots.release(); // Increment empty slots; wake up producer
        return item;
    }
};
```

---

## 3. Lock-Free Ring Buffer Bounded Queue (Disruptor Concept)

Uses a ring-buffer with atomic ticket sequences. Each cell tracks its own version counter. A cell can only be written to if its version matches `sequence`, and can only be read if its version matches `sequence + 1`. This allows multi-producer multi-consumer operations with zero mutexes.

```cpp
#include <atomic>
#include <vector>

template <typename T>
class LockFreeRingBuffer {
    struct Cell {
        T data;
        std::atomic<std::size_t> sequence;
    };

    std::vector<Cell> buffer;
    std::size_t bufferMask;
    alignas(64) std::atomic<std::size_t> enqueuePos{0};
    alignas(64) std::atomic<std::size_t> dequeuePos{0};

public:
    explicit LockFreeRingBuffer(std::size_t cap) : buffer(cap), bufferMask(cap - 1) {
        // Capacity must be a power of 2
        for (std::size_t i = 0; i < cap; ++i) {
            buffer[i].sequence.store(i, std::memory_order_relaxed);
        }
    }

    void push(const T& item) {
        std::size_t pos = enqueuePos.load(std::memory_order_relaxed);
        while (true) {
            Cell* cell = &buffer[pos & bufferMask];
            std::size_t seq = cell->sequence.load(std::memory_order_acquire);
            intdiff_t difference = static_cast<intdiff_t>(seq) - static_cast<intdiff_t>(pos);
            
            if (difference == 0) {
                // Try to claim the enqueue position
                if (enqueuePos.compare_exchange_weak(pos, pos + 1, std::memory_order_relaxed)) {
                    cell->data = item;
                    cell->sequence.store(pos + 1, std::memory_order_release);
                    return;
                }
            } else if (difference < 0) {
                // Buffer is full, reload and retry
                pos = enqueuePos.load(std::memory_order_relaxed);
            } else {
                pos = enqueuePos.load(std::memory_order_relaxed);
            }
        }
    }

    bool pop(T& item) {
        std::size_t pos = dequeuePos.load(std::memory_order_relaxed);
        while (true) {
            Cell* cell = &buffer[pos & bufferMask];
            std::size_t seq = cell->sequence.load(std::memory_order_acquire);
            intdiff_t difference = static_cast<intdiff_t>(seq) - static_cast<intdiff_t>(pos + 1);
            
            if (difference == 0) {
                // Try to claim the dequeue position
                if (dequeuePos.compare_exchange_weak(pos, pos + 1, std::memory_order_relaxed)) {
                    item = cell->data;
                    cell->sequence.store(pos + bufferMask + 1, std::memory_order_release);
                    return true;
                }
            } else if (difference < 0) {
                // Buffer is empty, reload and return false (or loop retry depending on blocking requirements)
                return false; 
            } else {
                pos = dequeuePos.load(std::memory_order_relaxed);
            }
        }
    }
};
```

---

*<- [[08_CPS_Read_Write_Lock|Read-Write Locks]] · [[08_CPS_Thread_Pool|Thread Pools ->]]*
