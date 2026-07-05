---
tags: [cpp, exceptions, noexcept, vector-optimization, exception-guarantees]
links: ["[[11_Exc_Index]]", "[[11_Exc_Exceptions_vs_Error_Codes]]", "[[11_Exc_Exception_Safety_Guarantees]]"]
---

# Exception Handling & Safety -- Noexcept & Specifications

*<- [[11_Exc_Exceptions_vs_Error_Codes|Exceptions vs Error Codes]] · [[11_Exc_Exception_Safety_Guarantees|Exception Safety Guarantees ->]]*

---

The **`noexcept`** specifier declares whether a function can throw exceptions, enabling compiler optimizations.

---

## 1. The `noexcept` Specifier & Crash Rule

Marking a function `noexcept` guarantees it will not throw.
- **Optimization**: The compiler does not generate stack unwinding tables for the function, reducing binary size and allowing aggressive inlining.
- **The Crash Rule**: If a function marked `noexcept` throws an exception anyway, the runtime immediately calls **`std::terminate()`**, crashing the program. It does **not** search for outer catch blocks.

```cpp
#include <iostream>
#include <exception>

void badFunction() noexcept {
    throw std::runtime_error("Oops"); // CRASH: Calls std::terminate() immediately!
}
```

---

## 2. The `noexcept` Operator (Compile-Time)

The `noexcept(expression)` operator checks at compile-time whether an expression is guaranteed not to throw. It returns a boolean.

```cpp
#include <type_traits>

template <typename T>
void swap(T& a, T& b) noexcept(noexcept(a.swap(b))) {
    a.swap(b);
}
```

---

## 3. The `std::vector` Reallocation Optimization Trap

A critical performance pitfall in C++ involves `std::vector` and custom classes.
- When a `std::vector` exceeds its capacity, it reallocates memory and relocates existing elements to the new memory block.
- **The Condition**: `std::vector` checks if the element's move constructor is marked **`noexcept`** (using `std::is_nothrow_move_constructible`).
- **If `noexcept`**: The vector **moves** elements to the new block ($O(1)$ pointer swaps per element).
- **If NOT `noexcept`**: The vector falls back to **copying** all elements to the new block! This degrades performance from pointer moves to full allocations.

### Why does this happen? (Strong Exception Guarantee)
If a move constructor throws an exception in the middle of a vector reallocation, the source vector is already partially destroyed/looted. It is impossible to roll back to the original state.
To preserve the **Strong Exception Guarantee** (the state of the vector must remain unchanged if an operation fails), `std::vector` must copy elements instead. If a copy constructor throws, the source vector is untouched, enabling safe rollback.

```cpp
#include <vector>
#include <type_traits>
#include <iostream>

class VectorSafe {
public:
    VectorSafe() = default;
    
    // Explicitly marked noexcept
    VectorSafe(VectorSafe&&) noexcept {
        std::cout << "Move constructor\n";
    }
};

class VectorUnsafe {
public:
    VectorUnsafe() = default;
    
    // Missing noexcept!
    VectorUnsafe(VectorUnsafe&&) {
        std::cout << "Copy constructor\n";
    }
};

void demoVectorRealloc() {
    std::cout << std::boolalpha;
    std::cout << "Is VectorSafe nothrow move constructible? " 
              << std::is_nothrow_move_constructible_v<VectorSafe> << "\n"; // true
              
    std::cout << "Is VectorUnsafe nothrow move constructible? " 
              << std::is_nothrow_move_constructible_v<VectorUnsafe> << "\n"; // false
              
    std::vector<VectorSafe> safeVec(1);
    std::cout << "Reallocating safeVec:\n";
    safeVec.emplace_back(); // Triggers move constructor!

    std::vector<VectorUnsafe> unsafeVec(1);
    std::cout << "Reallocating unsafeVec:\n";
    // unsafeVec.emplace_back(); // Falls back to copy operations (runs copy constructor)!
}
```

---

*<- [[11_Exc_Exceptions_vs_Error_Codes|Exceptions vs Error Codes]] · [[11_Exc_Exception_Safety_Guarantees|Exception Safety Guarantees ->]]*
