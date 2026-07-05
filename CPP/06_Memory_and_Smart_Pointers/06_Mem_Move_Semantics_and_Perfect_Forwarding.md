---
tags: [cpp, memory, move-semantics, std-move, perfect-forwarding, reference-collapsing]
links: ["[[06_Mem_Index]]", "[[06_Mem_Shared_Ptr_and_Weak_Ptr]]", "[[06_Mem_Custom_Deleters_and_Allocators]]"]
---

# Memory & Smart Pointers -- Move Semantics & Perfect Forwarding

*<- [[06_Mem_Shared_Ptr_and_Weak_Ptr|Shared & Weak Pointers]] · [[06_Mem_Custom_Deleters_and_Allocators|Custom Deleters & Allocators ->]]*

---

## 1. What is `std::move` Under the Hood?

`std::move` does **not** move any bytes at runtime and generates no CPU instructions. 
- It is simply a compile-time **static_cast to an rvalue reference**:
  ```cpp
  template <typename T>
  typename std::remove_reference<T>::type&& move(T&& param) {
      using ReturnType = typename std::remove_reference<T>::type&&;
      return static_cast<ReturnType>(param);
  }
  ```
- It tells the compiler: "Treat this expression as an rvalue, allowing its resources to be hijacked."

---

## 2. Universal References & Reference Collapsing

A **Universal Reference** (or **Forwarding Reference**) is a reference parameter declared as `T&&` where `T` is a **deduced type** (e.g. function templates).
- It can bind to both lvalues and rvalues.

```cpp
template <typename T>
void process(T&& arg); // 'arg' is a forwarding reference
```

### Reference Collapsing Rules:
If the argument passed to `T&&` is an lvalue (`X&`), `T` is deduced as `X&`. The reference becomes `X& &&`. The compiler collapses references according to these rules:

1. `& &` $\to$ **`&`**
2. `& &&` $\to$ **`&`**
3. `&& &` $\to$ **`&`**
4. `&& &&` $\to$ **`&&`**

Only a double rvalue reference collapses to an rvalue reference. In all other cases, it collapses to an lvalue reference.

---

## 3. Perfect Forwarding using `std::forward`

When writing wrapper functions (like factory templates), we want to forward parameters to another function while **preserving their original value categories** (lvalue remains lvalue, rvalue remains rvalue).

- If we pass `arg` directly, it is a named variable, so it becomes an **lvalue**.
- **`std::forward<T>(arg)`** casts `arg` back to `T&&`. Thanks to reference collapsing, this restores its original value category!

```cpp
#include <utility>
#include <iostream>

void target(int& x) { std::cout << "Lvalue target\n"; }
void target(int&& x) { std::cout << "Rvalue target\n"; }

// Wrapper function template
template <typename T>
void wrapper(T&& arg) {
    // std::forward restores the original value category of arg
    target(std::forward<T>(arg)); 
}

void demoForwarding() {
    int val = 42;
    wrapper(val);          // Passes lvalue -> outputs: Lvalue target
    wrapper(std::move(val)); // Passes rvalue -> outputs: Rvalue target
}
```

---

*<- [[06_Mem_Shared_Ptr_and_Weak_Ptr|Shared & Weak Pointers]] · [[06_Mem_Custom_Deleters_and_Allocators|Custom Deleters & Allocators ->]]*
