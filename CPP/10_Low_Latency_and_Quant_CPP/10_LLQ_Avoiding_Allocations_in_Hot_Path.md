---
tags: [cpp, low-latency, hot-path, allocation-audits, operator-new-overload]
links: ["[[10_LLQ_Index]]", "[[10_LLQ_Memory_Pools_and_Arena_Allocators]]", "[[10_LLQ_Fixed_Point_vs_Floating_Point]]"]
---

# Low-Latency & Quant C++ -- Avoiding Allocations in Hot Path

*<- [[10_LLQ_Memory_Pools_and_Arena_Allocators|Memory Pools & Arena Allocators]] · [[10_LLQ_Fixed_Point_vs_Floating_Point|Fixed-Point vs Floating-Point ->]]*

---

In low-latency systems (e.g. HFT feed handlers or execution engines), the **Hot Path** is the sequence of instructions executed on critical events (like receiving a market packet and sending an order). 
The golden rule of low-latency programming is: **Zero dynamic heap allocations in the hot path.**

---

## 1. Hot Path Constraints Checklist

- **No `new`/`delete` or `malloc`/`free`**: Handled via pre-allocated memory pools or stack arenas.
- **No Hidden Allocations**: Avoid appending to `std::string` (can exceed SSO limit and allocate), resizing `std::vector` (causes reallocation), or capturing large variables in lambda closures.
- **No Blocking or Mutex Locks**: Mutex acquisition failure triggers a kernel context switch, adding $\sim 2\text{--}5\mu\text{s}$ of latency.
- **No Synchronous I/O or Logging**: Network packets and file writes must be handed off to a lock-free background thread.

---

## 2. Auditing Dynamic Allocations at Runtime

To guarantee that no developer accidentally introduces a heap allocation in the hot path, we can overload the global **`operator new`** and **`operator delete`** to check an active thread-local flag. 
If any heap allocation occurs while the hot path flag is enabled, the program immediately aborts or throws an exception.

```cpp
#include <cstddef>
#include <new>
#include <iostream>
#include <stdexcept>

// Thread-local flag checking if the current thread is executing in the hot path
inline thread_local bool threadInHotPath = false;

struct HotPathScope {
    HotPathScope() { threadInHotPath = true; }
    ~HotPathScope() { threadInHotPath = false; }
};

// Overload global operator new
void* operator new(std::size_t size) {
    if (threadInHotPath) {
        // Crash immediately or throw to detect hot path leaks during test runs!
        std::cerr << "CRITICAL ERROR: Dynamic memory allocation inside hot path! Size: " << size << " bytes.\n";
        std::abort(); 
    }
    void* ptr = std::malloc(size);
    if (!ptr) throw std::bad_alloc();
    return ptr;
}

void operator delete(void* ptr) noexcept {
    if (threadInHotPath) {
        std::cerr << "CRITICAL ERROR: Dynamic memory deallocation inside hot path!\n";
        std::abort();
    }
    std::free(ptr);
}

// Demo usage
void hotPathFunction() {
    // Simulated work
    int* val = new int(10); // CRASHES the program!
}

void demoAudit() {
    std::cout << "Starting initialization...\n";
    int* p = new int(5); // OK: not in hot path yet
    delete p;

    std::cout << "Entering Hot Path...\n";
    {
        HotPathScope scope; // Sets threadInHotPath to true
        hotPathFunction(); 
    }
}
```

---

*<- [[10_LLQ_Memory_Pools_and_Arena_Allocators|Memory Pools & Arena Allocators]] · [[10_LLQ_Fixed_Point_vs_Floating_Point|Fixed-Point vs Floating-Point ->]]*
