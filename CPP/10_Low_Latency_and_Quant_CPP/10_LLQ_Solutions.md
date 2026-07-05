---
tags: [cpp, low-latency, solutions]
links: ["[[10_LLQ_Index]]", "[[10_LLQ_Problems]]", "[[10_LLQ_Tricky]]"]
---

# Low-Latency & Quant C++ -- Solutions

*<- [[10_LLQ_Problems|Problems]] · [[10_LLQ_Tricky|Tricky Low-Latency Gotchas ->]]*

---

### Solution 1: Lock-Free SPSC Queue Ring Buffer

```cpp
#include <atomic>
#include <new>

template <typename T, std::size_t Capacity = 1024>
class LockFreeSPSC {
    // Separate indices by 64 bytes (standard cache line size) to prevent false sharing
    alignas(64) std::atomic<std::size_t> writeIdx{0};
    alignas(64) std::atomic<std::size_t> readIdx{0};
    
    T buffer[Capacity];

public:
    LockFreeSPSC() = default;

    bool push(const T& val) {
        std::size_t currWrite = writeIdx.load(std::memory_order_relaxed);
        std::size_t currRead = readIdx.load(std::memory_order_acquire); // Acquire latest read index

        if (currWrite - currRead >= Capacity) {
            return false; // Queue is full
        }

        buffer[currWrite % Capacity] = val;
        // Release: ensures data write is complete before updating write index
        writeIdx.store(currWrite + 1, std::memory_order_release); 
        return true;
    }

    bool pop(T& val) {
        std::size_t currRead = readIdx.load(std::memory_order_relaxed);
        std::size_t currWrite = writeIdx.load(std::memory_order_acquire); // Acquire latest write index

        if (currRead == currWrite) {
            return false; // Queue is empty
        }

        val = buffer[currRead % Capacity];
        // Release: ensures data read is complete before updating read index
        readIdx.store(currRead + 1, std::memory_order_release);
        return true;
    }
};
```

---

### Solution 2: Fixed-Size Block Memory Pool

```cpp
#include <cstddef>
#include <new>
#include <utility>
#include <stdexcept>

template <typename T, std::size_t BlockCount = 1024>
class FixedBlockPool {
    union Node {
        char elementStorage[sizeof(T)];
        Node* next;
    };

    Node pool[BlockCount];
    Node* freeList = nullptr;

public:
    FixedBlockPool() {
        for (std::size_t i = 0; i < BlockCount - 1; ++i) {
            pool[i].next = &pool[i + 1];
        }
        pool[BlockCount - 1].next = nullptr;
        freeList = &pool[0];
    }

    template <typename... Args>
    T* allocate(Args&&... args) {
        if (!freeList) {
            throw std::bad_alloc(); // Pool is full
        }

        Node* block = freeList;
        freeList = freeList->next;

        T* obj = reinterpret_cast<T*>(block->elementStorage);
        // Placement new construction
        new (obj) T(std::forward<Args>(args)...); 
        return obj;
    }

    void deallocate(T* ptr) {
        if (!ptr) return;

        ptr->~T(); // Call destructor

        Node* block = reinterpret_cast<Node*>(ptr);
        block->next = freeList;
        freeList = block; // Push back to free list
    }
};
```

---

### Solution 3: Hot-Path Memory Allocation Auditor

```cpp
#include <cstddef>
#include <new>
#include <iostream>
#include <cstdlib>

// Thread-local auditing flag
inline thread_local bool threadInHotPath = false;

class HotPathAuditor {
public:
    HotPathAuditor() { threadInHotPath = true; }
    ~HotPathAuditor() { threadInHotPath = false; }
};

// Global Operator Overloads
void* operator new(std::size_t size) {
    if (threadInHotPath) {
        std::cerr << "CRITICAL HOT-PATH VIOLATION: Memory allocated! Size: " << size << " bytes.\n";
        std::abort(); // Force termination immediately
    }
    void* ptr = std::malloc(size);
    if (!ptr) throw std::bad_alloc();
    return ptr;
}

void operator delete(void* ptr) noexcept {
    if (threadInHotPath) {
        std::cerr << "CRITICAL HOT-PATH VIOLATION: Memory deallocated!\n";
        std::abort();
    }
    std::free(ptr);
}
```

---

### Solution 4: Fixed-Point Currency Class

```cpp
#include <cstdint>
#include <cmath>

class CurrencyDec4 {
    std::int64_t rawValue = 0;
    static constexpr std::int64_t SCALE = 10000;

    explicit CurrencyDec4(std::int64_t rawVal, bool) : rawValue(rawVal) {}

public:
    CurrencyDec4() = default;
    
    explicit CurrencyDec4(double val) {
        rawValue = static_cast<std::int64_t>(val * SCALE + (val >= 0 ? 0.5 : -0.5));
    }

    CurrencyDec4 operator+(const CurrencyDec4& other) const {
        return CurrencyDec4(rawValue + other.rawValue, true);
    }

    CurrencyDec4 operator-(const CurrencyDec4& other) const {
        return CurrencyDec4(rawValue - other.rawValue, true);
    }

    CurrencyDec4 operator*(const CurrencyDec4& other) const {
        return CurrencyDec4((rawValue * other.rawValue) / SCALE, true);
    }

    double toDouble() const {
        return static_cast<double>(rawValue) / SCALE;
    }
};
```

---

*<- [[10_LLQ_Problems|Problems]] · [[10_LLQ_Tricky|Tricky Low-Latency Gotchas ->]]*
