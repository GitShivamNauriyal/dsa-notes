---
tags: [cpp, memory, move-semantics, std-move, perfect-forwarding, std-forward, reference-collapsing]
links: ["[[06_Mem_Index]]", "[[06_Mem_Shared_Ptr_and_Weak_Ptr]]", "[[06_Mem_Custom_Deleters_and_Allocators]]"]
---

# Memory & Smart Pointers -- Move Semantics & Perfect Forwarding

*<- [[06_Mem_Shared_Ptr_and_Weak_Ptr|Shared & Weak Pointers]] · [[06_Mem_Custom_Deleters_and_Allocators|Custom Deleters & Allocators ->]]*

---

## 1. What is `std::move` Under the Hood?

`std::move` does **not** move any bytes or generate assembly code at runtime. It is purely a compile-time cast.

### The Standard Implementation
```cpp
template <typename T>
typename std::remove_reference<T>::type&& move(T&& param) {
    using ReturnType = typename std::remove_reference<T>::type&&;
    return static_cast<ReturnType>(param);
}
```
- **Action**: Casts its argument to an rvalue reference (`T&&`). This signals to the compiler that the object is temporary and its resources can be safely stolen.

### The Raw Pointer Move Gotcha:
Moving a raw pointer or primitive variable **does not clear the source**! 
- If you move a raw pointer, the compiler merely performs a value copy of the address.
- You **must** manually null out the source pointer in your move constructor to prevent double-free crashes.

```cpp
class Widget {
    int* data;
public:
    Widget(Widget&& other) noexcept : data(other.data) {
        other.data = nullptr; // Mandatory: otherwise, other's destructor deletes the memory!
    }
};
```

---

## 2. Universal References & Reference Collapsing

A **Universal Reference** (or **Forwarding Reference**) is declared as `T&&` where `T` is a **deduced type** (e.g. inside a function template: `template <typename T> void func(T&& arg);`).

### Reference Collapsing Rules:
If an lvalue of type `X` is passed, `T` is deduced as `X&`. The parameter becomes `X& &&`. The compiler collapses these multiple references to a single reference:

1. `& &` $\to$ **`&`**
2. `& &&` $\to$ **`&`**
3. `&& &` $\to$ **`&`**
4. `&& &&` $\to$ **`&&`**

- Only a double rvalue reference collapses to an rvalue reference; all other combinations collapse to a standard lvalue reference.

---

## 3. Perfect Forwarding using `std::forward`

When writing generic wrapper functions (like `std::make_unique` or factory functions), we must forward arguments to another function while **preserving their original value categories** (lvalues stay lvalues, rvalues stay rvalues).
- Because arguments inside the wrapper have names, they are treated as **lvalues** by default.
- **`std::forward`** restores the original value category using reference collapsing.

### The Standard Implementation of `std::forward`
```cpp
// Overload 1: For lvalue arguments
template <typename T>
constexpr T&& forward(std::remove_reference_t<T>& arg) noexcept {
    return static_cast<T&&>(arg);
}

// Overload 2: For rvalue arguments (prevents forwarding rvalues as lvalues)
template <typename T>
constexpr T&& forward(std::remove_reference_t<T>&& arg) noexcept {
    static_assert(!std::is_lvalue_reference_v<T>, "Cannot forward rvalue as lvalue");
    return static_cast<T&&>(arg);
}
```

### Trace of Reference Collapsing with `std::forward`:
Let's see what happens when calling `wrapper(T&& arg)` with `std::forward<T>(arg)`:

- **Case A: Lvalue passed (`int x; wrapper(x);`)**:
  - `T` is deduced as `int&`.
  - `std::forward<int&>(arg)` returns `static_cast<int& &&>(arg)`.
  - Reference Collapsing yields: **`int&`** (lvalue reference preserved!).
- **Case B: Rvalue passed (`wrapper(10);`)**:
  - `T` is deduced as `int`.
  - `std::forward<int>(arg)` returns `static_cast<int&&>(arg)`.
  - Reference Collapsing yields: **`int&&`** (rvalue reference preserved!).

```cpp
#include <utility>
#include <iostream>

void target(int& x) { std::cout << "Lvalue target\n"; }
void target(int&& x) { std::cout << "Rvalue target\n"; }

template <typename T>
void wrapper(T&& arg) {
    target(std::forward<T>(arg)); // Perfect Forwarding!
}
```

---

*<- [[06_Mem_Shared_Ptr_and_Weak_Ptr|Shared & Weak Pointers]] · [[06_Mem_Custom_Deleters_and_Allocators|Custom Deleters & Allocators ->]]*
