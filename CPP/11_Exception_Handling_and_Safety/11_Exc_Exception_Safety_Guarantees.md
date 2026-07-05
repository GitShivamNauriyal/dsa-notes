---
tags: [cpp, exceptions, safety-guarantees, strong-exception-safety, copy-and-swap]
links: ["[[11_Exc_Index]]", "[[11_Exc_Noexcept_and_Specifications]]", "[[11_Exc_Stack_Unwinding_and_RAII]]"]
---

# Exception Handling & Safety -- Exception Safety Guarantees

*<- [[11_Exc_Noexcept_and_Specifications|Noexcept & Specifications]] · [[11_Exc_Stack_Unwinding_and_RAII|Stack Unwinding & RAII ->]]*

---

C++ defines three levels of exception safety guarantees when a function throws an exception.

---

## 1. The Three Safety Levels

### A. Basic Guarantee
- **Invariant**: If an exception is thrown, the program remains in a valid state. No memory is leaked, and no resources are left in corrupted states.
- **State**: The data inside the objects **can be modified**, meaning the state of the program might have changed from what it was before the function call.

### B. Strong Guarantee (Transactional)
- **Invariant**: If an exception is thrown, the state of the program is **rolled back** to exactly what it was before the function call.
- **State**: Either the operation completes successfully, or it has no effect (all or nothing).

### C. No-throw / No-fail Guarantee (`noexcept`)
- **Invariant**: The function is guaranteed to never throw an exception.
- **Requirement**: Destructors, move constructors, and swap operations **must** satisfy this guarantee to allow safe memory deallocation.

---

## 2. Implementing the Strong Guarantee (Copy-and-Swap Idiom)

The standard way to implement the **Strong Exception Guarantee** on assignment operators and modifying functions is the **Copy-and-Swap** idiom.

### Step-by-Step Copy-and-Swap Process:
1. Pass the parameter **by value** to trigger a copy construction. If the copy fails (e.g. out of memory), the exception is thrown *before* modifying `*this`.
2. Perform a **`noexcept` swap** to swap the contents of the temporary copy with the current object (`*this`).
3. The temporary copy goes out of scope, safely destroying the old resources in its destructor.

```cpp
#include <utility>
#include <cstddef>
#include <algorithm>

class SafeArray {
    int* data = nullptr;
    std::size_t sz = 0;

public:
    SafeArray() = default;
    
    // Copy Constructor
    SafeArray(const SafeArray& other) : sz(other.sz) {
        if (sz > 0) {
            data = new int[sz];
            std::copy(other.data, other.data + sz, data);
        }
    }

    // Destructor (No-throw guarantee)
    ~SafeArray() noexcept {
        delete[] data;
    }

    // No-throw Swap function
    void swap(SafeArray& other) noexcept {
        using std::swap;
        swap(data, other.data);
        swap(sz, other.sz);
    }

    // Assignment Operator implementing the Strong Exception Guarantee!
    SafeArray& operator=(SafeArray other) { // 1. Copy constructed by value
        swap(other); // 2. Swap state (No-throw)
        return *this;
    } // 3. 'other' goes out of scope and deallocates the old data
};
```

---

*<- [[11_Exc_Noexcept_and_Specifications|Noexcept & Specifications]] · [[11_Exc_Stack_Unwinding_and_RAII|Stack Unwinding & RAII ->]]*
