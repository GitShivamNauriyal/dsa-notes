---
tags: [cpp, performance, sso, sbo, stack-buffer, heap-avoidance]
links: ["[[09_Perf_Index]]", "[[09_Perf_Move_Semantics_for_Performance]]", "[[09_Perf_Problems]]"]
---

# Performance & Optimization -- Small Object Optimizations

*<- [[09_Perf_Move_Semantics_for_Performance|Move Semantics for Performance]] · [[09_Perf_Problems|Problems ->]]*

---

Dynamic allocations (`new`, `malloc`) are slow because they require searching the OS heap, locking memory managers, and updating heap page structures.
**Small Object Optimizations** bypass heap allocations entirely by utilizing internal stack buffers for small data sizes.

---

## 1. Small String Optimization (SSO)

In most standard library implementations, `std::string` uses **Small String Optimization (SSO)**.
- If a string's length is **15 characters or less** (on 64-bit platforms), it does **not** allocate memory on the heap.
- Instead, it stores the characters directly inside the `std::string` object itself (occupying the memory space usually reserved for the pointer to the heap block).

### Under the Hood Structure (ASCII Union Layout)
```
  std::string Layout Union (24 Bytes total):
  
  Case A: Large String (Allocated on Heap):
  ┌──────────────────────┬──────────────────────┬──────────────────────┐
  │ Pointer to Heap (8)  │ String Size (8)      │ Buffer Capacity (8)  │
  └──────────────────────┴──────────────────────┴──────────────────────┘
  
  Case B: Small String (Stored Inline):
  ┌─────────────────────────────────────────────────────────────┬──────┐
  │ Characters Array (15 Bytes)                                 │Size(1)
  └─────────────────────────────────────────────────────────────┴──────┘
```

Because it uses a `union`, the size of `std::string` remains 24 bytes in both cases.

---

## 2. Custom Small Buffer Optimization (SBO) Implementation

We can apply the Small Buffer Optimization to a custom collection class. If the count of items is within the threshold limit, we store them in a local stack-allocated array. If we exceed the limit, we allocate a heap block.

```cpp
#include <cstddef>
#include <algorithm>
#include <new>
#include <iostream>

template <typename T, std::size_t Threshold = 4>
class SBOBuffer {
    T* data = nullptr;
    std::size_t sz = 0;
    std::size_t cap = 0;

    // Stack storage for SBO
    alignas(T) char inlineStorage[Threshold * sizeof(T)];

    T* getInlinePtr() {
        return reinterpret_cast<T*>(inlineStorage);
    }

public:
    SBOBuffer() {
        data = getInlinePtr();
        cap = Threshold;
    }

    ~SBOBuffer() {
        for (std::size_t i = 0; i < sz; ++i) {
            data[i].~T();
        }
        if (data != getInlinePtr()) {
            delete[] reinterpret_cast<char*>(data);
        }
    }

    void push_back(const T& val) {
        if (sz >= cap) {
            // Allocate heap block
            std::size_t newCap = cap * 2;
            T* newBlock = reinterpret_cast<T*>(new char[newCap * sizeof(T)]);
            
            // Move old elements
            for (std::size_t i = 0; i < sz; ++i) {
                new (newBlock + i) T(std::move(data[i]));
                data[i].~T();
            }
            
            // Free old block if it was on the heap
            if (data != getInlinePtr()) {
                delete[] reinterpret_cast<char*>(data);
            }

            data = newBlock;
            cap = newCap;
        }

        // Construct new element in place
        new (data + sz) T(val);
        sz++;
    }

    std::size_t size() const { return sz; }
    bool isUsingHeap() const { return data != getInlinePtr(); }
};

void demoSBO() {
    SBOBuffer<int, 3> buffer;
    
    buffer.push_back(10);
    buffer.push_back(20);
    std::cout << std::boolalpha << buffer.isUsingHeap() << "\n"; // Outputs: false (Stack storage!)
    
    buffer.push_back(30);
    buffer.push_back(40); // Exceeds threshold limit of 3!
    std::cout << buffer.isUsingHeap() << "\n";                 // Outputs: true  (Heap allocated!)
}
```

---

*<- [[09_Perf_Move_Semantics_for_Performance|Move Semantics for Performance]] · [[09_Perf_Problems|Problems ->]]*
