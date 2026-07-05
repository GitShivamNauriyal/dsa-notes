---
tags: [cpp, core, const, constexpr, consteval, constinit]
links: ["[[02_Core_Index]]", "[[02_Core_Value_Categories]]", "[[02_Core_Storage_Duration_and_Linkage]]"]
---

# Core Language Mechanics -- Const & Compile-Time Evaluation

*<- [[02_Core_Value_Categories|Value Categories]] · [[02_Core_Storage_Duration_and_Linkage|Storage & Linkage ->]]*

---

## 1. Keywords Comparison Matrix

| Keyword | Evaluation Phase | Applies to | Meaning |
|---|---|---|---|
| **`const`** | Runtime (usually) | Variables, member functions | Read-only view of data. Variable cannot be mutated after initialization. |
| **`constexpr`** | Compile-time OR Runtime | Variables, functions | Variable/Function *can* be evaluated at compile-time if conditions allow. |
| **`consteval`** *(C++20)* | **Compile-time ONLY** | Functions | Immediate function. Must be evaluated at compile-time, else compilation fails. |
| **`constinit`** *(C++20)* | Compile-time initialization | Variables | Forces compile-time static initialization of variable. Variable remains mutable. |

---

## 2. Compile-Time vs Runtime Mechanics

### A. `constexpr` (Conditional Compile-Time)
A `constexpr` function can be run at either compile-time or runtime depending on how it's invoked:
- If all arguments are compile-time constants AND the output is assigned to a `constexpr` variable, evaluation happens at compile-time.
- If called with runtime variables, it behaves like an ordinary runtime function.

```cpp
constexpr int square(int x) {
    return x * x;
}

void demo() {
    constexpr int val = square(5); // Evaluated at compile-time
    
    int y = 5;
    int val2 = square(y);          // Evaluated at runtime (behaves like a normal function)
}
```

---

### B. `consteval` (C++20 Immediate Functions)
Unlike `constexpr`, a `consteval` function **guarantees** compile-time execution. If the arguments are not compile-time constants, it generates a compiler error.

```cpp
consteval int cube(int x) {
    return x * x * x;
}

void demoConsteval() {
    constexpr int c1 = cube(3); // OK: evaluated at compile-time
    
    int y = 3;
    // int c2 = cube(y);       // COMPILER ERROR: y is not a compile-time constant!
}
```

---

### C. `constinit` (C++20 Static Initialization Guarantee)
In C++, global variables defined across different files have undefined initialization order at runtime, leading to the **Static Initialization Order Fiasco** (where one global variable reads another uninitialized global).
- `constinit` guarantees that a global variable is initialized at compile-time.
- Unlike `constexpr` or `const`, `constinit` variables **can be modified** later at runtime.

```cpp
constinit int globalCounter = 100; // Guaranteed to be initialized at compile-time

void incrementGlobal() {
    globalCounter++; // OK: constinit variables are mutable!
}
```

---

### D. `if consteval` (C++23)
Enables branching depending on whether the function is currently executing in a compile-time context (allowing optimization like using faster intrinsics at runtime but falling back to pure C++ at compile-time).

```cpp
#include <type_traits>

constexpr double power(double base, int exp) {
    if consteval {
        // Run slow, simple compile-time compatible loop
        double res = 1.0;
        for (int i = 0; i < exp; i++) res *= base;
        return res;
    } else {
        // Run optimized assembly/intrinsics at runtime
        // e.g. return std::pow(base, exp);
        return base; 
    }
}
```

---

*<- [[02_Core_Value_Categories|Value Categories]] · [[02_Core_Storage_Duration_and_Linkage|Storage & Linkage ->]]*
