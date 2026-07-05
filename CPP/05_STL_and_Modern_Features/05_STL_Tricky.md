---
tags: [cpp, stl, tricky, dangling-views, lambda-captures, std-function]
links: ["[[05_STL_Index]]", "[[05_STL_Solutions]]", "[[../06_Memory_and_Smart_Pointers/06_Mem_Index]]"]
---

# STL & Modern Features -- Tricky & Higher-Order

*<- [[05_STL_Solutions|Solutions]] · [[../06_Memory_and_Smart_Pointers/06_Mem_Index|Memory & Smart Pointers ->]]*

---

## 1. Dangling References in Lazily-Evaluated Views

**The Trap**: Storing a view built over a temporary (rvalue) container.
- Because views are **lazily evaluated**, they do not copy the container's elements. They only store iterators/references to the underlying container.
- If the container is a temporary object returned by a function, it is destroyed at the end of the expression, leaving the view's internal pointers dangling!

```cpp
#include <vector>
#include <ranges>
#include <iostream>

std::vector<int> getTemps() {
    return {1, 2, 3, 4, 5};
}

void demoDanglingView() {
    // getTemps() returns a temporary vector by value.
    // The view stores references to this temporary vector.
    // The temporary vector is destroyed at the semicolon!
    auto evens = getTemps() | std::views::filter([](int x) { return x % 2 == 0; });
    
    // UB: Iterating over elements of a deallocated vector!
    // for (int x : evens) {
    //     std::cout << x << " "; 
    // }
}
```

---

## 2. `std::function` Performance Overhead

**The Trap**: Wrapping all callables (lambdas) in `std::function` blindly.
- **`std::function`**: A type-safe polymorphic wrapper. It does **type erasure** (allowing callables of different actual lambda closure classes to be stored in the same variable type).
- **The Overhead**:
  1. **Heap Allocation**: If the captured closure object size exceeds the Small Buffer Optimization (SBO) limit, `std::function` allocates memory on the heap.
  2. **Indirect Calls (Virtual Dispatch equivalent)**: Type erasure prevents the compiler from performing **inlining**. Calling `std::function` requires an indirect function pointer jump.
- **Rule of Thumb**: Use raw lambdas or template parameters directly to preserve zero-cost compile-time binding. Only use `std::function` when you need dynamic storage (e.g. storing a vector of callbacks).

---

## 3. Implicit `this` Capture Pitfall in Member Lambdas

**The Trap**: Writing `[=]` inside a class member function.
- Many developers assume `[=]` copies the member variables of the class by value.
- **Reality**: It captures the **`this` pointer** by value! Member variables are still accessed via `this->member`.
- If the enclosing object is destroyed, executing the lambda later dereferences a dangling pointer, causing crashes.

```cpp
#include <functional>
#include <iostream>

class Task {
    int data = 100;
public:
    std::function<void()> getCallback() {
        // [=] implicitly captures 'this' by value, NOT 'data'!
        return [=]() {
            std::cout << data << "\n"; // accesses this->data
        };
    }
};

std::function<void()> createDanglingCallback() {
    Task t;
    return t.getCallback();
} // Object 't' is destroyed here!

void demoCrash() {
    auto cb = createDanglingCallback();
    // cb(); // UB: Dereferences deallocated 'this' pointer!
}
```

### The C++17 Fix (`[*this]`)
Capture `*this` explicitly to create a local, copied instance of the object inside the closure.

```cpp
std::function<void()> getSafeCallback() {
    return [*this]() {
        std::cout << data << "\n"; // Safe: data accessed from copy
    };
}
```

---

## Recap of Tricky STL Gotchas

| Gotcha | Theme |
|---|---|
| **Dangling Views** | Views only store references; building views over temporary objects leaves their references dangling. |
| **`std::function` Overhead** | Performs type erasure which disables inlining, and can trigger heap allocations if captures exceed SBO. |
| **Implicit `this` Capture** | `[=]` inside classes captures `this` pointer by value; resolved via C++17 `[*this]` object copy. |

---

*<- [[05_STL_Solutions|Solutions]] · [[../06_Memory_and_Smart_Pointers/06_Mem_Index|Memory & Smart Pointers ->]]*
