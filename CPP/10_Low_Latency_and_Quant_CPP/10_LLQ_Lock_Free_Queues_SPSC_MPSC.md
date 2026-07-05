---
tags: [cpp, low-latency, lock-free, spsc, mpsc, ring-buffer]
links: ["[[10_LLQ_Index]]", "[[10_LLQ_Memory_Pools_and_Arena_Allocators]]"]
---

# Low-Latency & Quant C++ -- Lock-Free Queues (SPSC/MPSC)

*<- [[10_LLQ_Index|Index]] · [[10_LLQ_Memory_Pools_and_Arena_Allocators|Memory Pools & Arena Allocators ->]]*

---

Lock-free queues are the backbone of message passing in low-latency systems. 

---

## 1. Single-Producer Single-Consumer (SPSC) Queue

In an **SPSC Queue**, exactly one thread writes (produces) and exactly one thread reads (consumes).
- **The Optimization**: Because there is no contention on the read/write ends between multiple threads, we do **not** need expensive Compare-And-Swap (CAS) instructions.
- We only need atomic loads and stores with **acquire-release** memory ordering to ensure data visibility.
- This results in extremely high throughput and sub-nanosecond latencies.

### SPSC Implementation
```cpp
#include <atomic>
#include <vector>
#include <new>

template <typename T, std::size_t Capacity = 1024>
class LockFreeSPSC {
    // Force indices onto separate cache lines to prevent false sharing!
    alignas(64) std::atomic<std::size_t> writeIdx{0};
    alignas(64) std::atomic<std::size_t> readIdx{0};
    
    T buffer[Capacity];

public:
    LockFreeSPSC() = default;

    bool push(const T& val) {
        std::size_t currWrite = writeIdx.load(std::memory_order_relaxed);
        std::size_t currRead = readIdx.load(std::memory_order_acquire); // Acquire: check latest read

        // If buffer is full
        if (currWrite - currRead >= Capacity) {
            return false;
        }

        buffer[currWrite % Capacity] = val;
        // Release: ensure write is visible before incrementing writeIdx
        writeIdx.store(currWrite + 1, std::memory_order_release); 
        return true;
    }

    bool pop(T& val) {
        std::size_t currRead = readIdx.load(std::memory_order_relaxed);
        std::size_t currWrite = writeIdx.load(std::memory_order_acquire); // Acquire: check latest write

        // If buffer is empty
        if (currRead == currWrite) {
            return false;
        }

        val = buffer[currRead % Capacity];
        // Release: ensure pop is completed before incrementing readIdx
        readIdx.store(currRead + 1, std::memory_order_release);
        return true;
    }
};
```

---

## 2. Multi-Producer Single-Consumer (MPSC) Queue

In an **MPSC Queue**, multiple threads push data (producing) while a single dedicated thread pops (consuming).
- Common in **logging frameworks** (many worker threads submitting log events to a single file-writer thread) and **network engines**.
- **The Mechanic**: Producers contend for the write position, requiring atomic **Compare-And-Swap (CAS)** or `exchange` loops. The single consumer reads without contention, needing no CAS.

```cpp
#include <atomic>
#include <new>

template <typename T>
class LockFreeMPSC {
    struct Node {
        T data;
        std::atomic<Node*> next{nullptr};
        Node() = default;
        Node(const T& val) : data(val) {}
    };

    // Head is only accessed by the single consumer (pop)
    alignas(64) Node* head;
    
    // Tail is modified by multiple producers (push)
    alignas(64) std::atomic<Node*> tail;

public:
    LockFreeMPSC() {
        Node* dummy = new Node();
        head = dummy;
        tail.store(dummy);
    }

    ~LockFreeMPSC() {
        Node* curr = head;
        while (curr) {
            Node* next = curr->next.load(std::memory_order_relaxed);
            delete curr;
            curr = next;
        }
    }

    void push(const T& val) {
        Node* newNode = new Node(val);
        
        // Exchange tail atomically: returns the old tail and sets new tail to newNode.
        // Multiple producers loop inside exchange or succeed immediately.
        Node* prevTail = tail.exchange(newNode, std::memory_order_acq_rel);
        
        // Link the old tail to the new node
        prevTail->next.store(newNode, std::memory_order_release);
    }

    bool pop(T& val) {
        Node* currHead = head;
        Node* nextNode = currHead->next.load(std::memory_order_acquire);
        
        if (nextNode == nullptr) {
            return false; // Queue is empty
        }
        
        val = nextNode->data;
        head = nextNode; // Advance head (old head node is discarded)
        delete currHead;
        return true;
    }
};
```

---

*<- [[10_LLQ_Index|Index]] · [[10_LLQ_Memory_Pools_and_Arena_Allocators|Memory Pools & Arena Allocators ->]]*
