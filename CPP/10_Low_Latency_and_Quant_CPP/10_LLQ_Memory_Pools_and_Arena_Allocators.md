---
tags: [cpp, low-latency, memory-pool, arena-allocator, bump-allocator, fragmentations]
links: ["[[10_LLQ_Index]]", "[[10_LLQ_Lock_Free_Queues_SPSC_MPSC]]", "[[10_LLQ_Avoiding_Allocations_in_Hot_Path]]"]
---

# Low-Latency & Quant C++ -- Memory Pools & Arena Allocators

*<- [[10_LLQ_Lock_Free_Queues_SPSC_MPSC|Lock-Free Queues (SPSC/MPSC)]] · [[10_LLQ_Avoiding_Allocations_in_Hot_Path|Avoiding Allocations in Hot Path ->]]*

---

Standard OS allocations (`malloc`/`new`) introduce latency jitter and heap fragmentation. In high-frequency trading (HFT) and game loops, custom allocators are used to manage memory deterministically in $O(1)$ time.

---

## 1. Fixed-Size Block Memory Pool

A **Memory Pool** pre-allocates a large contiguous chunk of memory and divides it into equal-sized blocks.
- It maintains a **freelist** (a singly-linked list of unused blocks).
- **Allocation**: Pop a block from the freelist ($O(1)$).
- **Deallocation**: Push the block back onto the freelist ($O(1)$).
- **Result**: Zero heap fragmentation and guaranteed deterministic allocation speed.

```cpp
#include <cstddef>
#include <stdexcept>
#include <new>

template <typename T, std::size_t BlockCount = 1024>
class FixedMemoryPool {
    union Node {
        char elementStorage[sizeof(T)];
        Node* next; // Linked list pointer to track free blocks
    };

    Node pool[BlockCount];
    Node* freeList = nullptr;

public:
    FixedMemoryPool() {
        // Link all blocks in the pool together into a free list
        for (std::size_t i = 0; i < BlockCount - 1; ++i) {
            pool[i].next = &pool[i + 1];
        }
        pool[BlockCount - 1].next = nullptr;
        freeList = &pool[0];
    }

    template <typename... Args>
    T* allocate(Args&&... args) {
        if (!freeList) {
            throw std::bad_alloc(); // Pool is exhausted
        }

        // Pop from freelist
        Node* block = freeList;
        freeList = freeList->next;

        // Construct object in place inside the block
        T* obj = reinterpret_cast<T*>(block->elementStorage);
        new (obj) T(std::forward<Args>(args)...);
        return obj;
    }

    void deallocate(T* ptr) {
        if (!ptr) return;

        // Call destructor
        ptr->~T();

        // Push block back onto freelist
        Node* block = reinterpret_cast<Node*>(ptr);
        block->next = freeList;
        freeList = block;
    }
};
```

---

## 2. Arena Allocator (Bump Allocator)

An **Arena Allocator** pre-allocates a large contiguous buffer.
- ** Bump Allocation**: Allocating memory simply moves (bumps) an offset pointer forward by the requested size.
- **Deallocation constraint**: Individual objects cannot be deallocated. Instead, the entire arena's offset pointer is reset back to 0 at once (e.g. at the end of a transaction cycle or game frame).

```cpp
#include <cstddef>
#include <new>

class ArenaAllocator {
    char* buffer;
    std::size_t capacity;
    std::size_t offset = 0;

public:
    explicit ArenaAllocator(std::size_t size) : capacity(size) {
        buffer = new char[size];
    }

    ~ArenaAllocator() {
        delete[] buffer;
    }

    void* allocate(std::size_t size, std::size_t alignment = alignof(std::max_align_t)) {
        // Calculate padding needed to satisfy alignment
        std::size_t currentAddress = reinterpret_cast<std::size_t>(buffer + offset);
        std::size_t alignedAddress = (currentAddress + alignment - 1) & ~(alignment - 1);
        std::size_t newOffset = alignedAddress - reinterpret_cast<std::size_t>(buffer) + size;

        if (newOffset > capacity) {
            throw std::bad_alloc(); // Out of memory
        }

        offset = newOffset;
        return reinterpret_cast<void*>(alignedAddress);
    }

    // Reset the offset pointer back to 0 (clears all allocations in O(1))
    void reset() noexcept {
        offset = 0;
    }
    
    // Prevent copies
    ArenaAllocator(const ArenaAllocator&) = delete;
    ArenaAllocator& operator=(const ArenaAllocator&) = delete;
};
```

---

*<- [[10_LLQ_Lock_Free_Queues_SPSC_MPSC|Lock-Free Queues (SPSC/MPSC)]] · [[10_LLQ_Avoiding_Allocations_in_Hot_Path|Avoiding Allocations in Hot Path ->]]*
