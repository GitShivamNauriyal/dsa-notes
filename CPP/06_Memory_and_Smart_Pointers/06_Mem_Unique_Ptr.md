---
tags: [cpp, memory, unique-ptr, exclusive-ownership, make-unique]
links: ["[[06_Mem_Index]]", "[[06_Mem_Raw_Pointer_Pitfalls]]", "[[06_Mem_Shared_Ptr_and_Weak_Ptr]]"]
---

# Memory & Smart Pointers -- Unique Pointers

*<- [[06_Mem_Raw_Pointer_Pitfalls|Raw Pointer Pitfalls]] · [[06_Mem_Shared_Ptr_and_Weak_Ptr|Shared & Weak Pointers ->]]*

---

## 1. Exclusive Ownership Model

`std::unique_ptr` manages a heap object with **exclusive ownership**.
- Only one `std::unique_ptr` can point to a resource at any given time.
- **Non-Copyable**: Copy constructor and copy assignment operator are explicitly `= delete`d to prevent duplicate ownership.
- **Movable**: Ownership can be transferred to another `std::unique_ptr` using `std::move`.

```cpp
#include <memory>
#include <utility>

void demoUnique() {
    auto p1 = std::make_unique<int>(10);
    
    // std::unique_ptr<int> p2 = p1; // COMPILER ERROR: copy operation is deleted!
    
    // Transfer ownership using std::move
    std::unique_ptr<int> p2 = std::move(p1); // p1 is now nullptr, p2 owns the resource
}
```

---

## 2. Size & Performance: Zero-Cost Abstraction

A default `std::unique_ptr` is a **zero-cost abstraction**. 
- It compiles down to the exact same assembly as a raw pointer.
- Memory size is identical: `sizeof(std::unique_ptr<T>) == sizeof(T*)` (typically 8 bytes on 64-bit systems).

---

## 3. Custom Deleters

You can supply a custom deleter function/functor to control how the resource is cleaned up when `std::unique_ptr` goes out of scope (e.g. closing a file handle `fclose` instead of calling `delete`).

> [!WARNING]
> If you pass a stateful functor or lambda as a custom deleter, the size of the `std::unique_ptr` will increase to store that state, losing its zero-overhead property.

```cpp
#include <memory>
#include <cstdio>

// Custom deleter functor
struct FileCloser {
    void operator()(FILE* fp) const {
        if (fp) std::fclose(fp);
    }
};

void demoCustomDeleter() {
    // Pass custom deleter class in template parameter
    std::unique_ptr<FILE, FileCloser> uptr(std::fopen("test.txt", "r"));
    
    // When uptr goes out of scope, FileCloser::operator() is automatically called!
}
```

---

*<- [[06_Mem_Raw_Pointer_Pitfalls|Raw Pointer Pitfalls]] · [[06_Mem_Shared_Ptr_and_Weak_Ptr|Shared & Weak Pointers ->]]*
