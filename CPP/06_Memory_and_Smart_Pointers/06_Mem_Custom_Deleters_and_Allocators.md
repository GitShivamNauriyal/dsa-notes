---
tags: [cpp, memory, custom-deleters, custom-allocators, type-erasure]
links: ["[[06_Mem_Index]]", "[[06_Mem_Move_Semantics_and_Perfect_Forwarding]]", "[[06_Mem_Problems]]"]
---

# Memory & Smart Pointers -- Custom Deleters & Allocators

*<- [[06_Mem_Move_Semantics_and_Perfect_Forwarding|Move & Perfect Forwarding]] · [[06_Mem_Problems|Problems ->]]*

---

## 1. Custom Deleter Signature Differences

The way custom deleters are stored and matched differs fundamentally between `std::unique_ptr` and `std::shared_ptr`:

---

### A. `std::unique_ptr` (Compile-Time Binding)
- The deleter type is **part of the class template type signature**:
  `std::unique_ptr<T, Deleter>`
- **Advantage**: Zero runtime overhead. The compiler can inline the deleter call directly.
- **Disadvantage**: You cannot store unique pointers with different deleters in the same container (e.g. `std::vector<std::unique_ptr<int>>` cannot hold pointers using custom deleters).

```cpp
#include <memory>

struct CustomDeleter {
    void operator()(int* ptr) const { delete ptr; }
};

void demoUniqueSignature() {
    std::unique_ptr<int, CustomDeleter> p1(new int(10));
}
```

---

### B. `std::shared_ptr` (Runtime Type-Erasure)
- The deleter is passed as a **constructor argument** at runtime:
  `std::shared_ptr<T> p(new T, custom_deleter);`
- **Advantage**: The deleter type is **erased** from the class signature. You can mix different deleters in the same collection (e.g., `std::vector<std::shared_ptr<int>>`).
- **Disadvantage**: The deleter must be stored in the heap-allocated **control block**, adding dynamic allocation overhead if not optimized.

```cpp
#include <memory>
#include <vector>

void customFree(int* ptr) { delete ptr; }

void demoSharedSignature() {
    // Type is simply std::shared_ptr<int> regardless of the deleter!
    std::shared_ptr<int> p1(new int(10), customFree);
    std::shared_ptr<int> p2(new int(20)); // default delete
    
    // Store both in the same vector
    std::vector<std::shared_ptr<int>> vec;
    vec.push_back(p1);
    vec.push_back(p2);
}
```

---

## 2. Custom Allocators

By default, smart pointers allocate their control block or objects using `std::allocator`, which requests memory from the OS heap. In low-latency systems (e.g. HFT or games), OS allocation calls introduce jitter. We can supply a custom allocator to allocate memory from a pre-allocated stack arena or a **memory pool**.

- **`std::allocate_shared`**: Instantiates a shared pointer using a custom allocator to allocate both the control block and the object in a single contiguous block.

```cpp
#include <memory>
#include <vector>

// Dummy Custom Allocator Concept
template <typename T>
struct ArenaAllocator {
    using value_type = T;
    ArenaAllocator() = default;
    template <class U> constexpr ArenaAllocator(const ArenaAllocator<U>&) noexcept {}

    T* allocate(std::size_t n) {
        return static_cast<T*>(::operator new(n * sizeof(T)));
    }
    void deallocate(T* p, std::size_t) noexcept {
        ::operator delete(p);
    }
};

void demoAllocator() {
    ArenaAllocator<int> myAlloc;
    
    // Allocate shared pointer using custom allocator
    auto ptr = std::allocate_shared<int>(myAlloc, 100);
}
```

---

*<- [[06_Mem_Move_Semantics_and_Perfect_Forwarding|Move & Perfect Forwarding]] · [[06_Mem_Problems|Problems ->]]*
