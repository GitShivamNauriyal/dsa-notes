---
tags: [cpp, core, const, constexpr, consteval, constinit, siof, memory-residency]
links: ["[[02_Core_Index]]", "[[02_Core_Value_Categories]]", "[[02_Core_Storage_Duration_and_Linkage]]"]
---

# Core Language Mechanics -- Const & Compile-Time Evaluation

*<- [[02_Core_Value_Categories|Value Categories]] · [[02_Core_Storage_Duration_and_Linkage|Storage & Linkage ->]]*

---

## 1. Keywords Comparison Matrix

| Keyword | Evaluation Phase | Applies to | Meaning | Memory Residency |
|---|---|---|---|---|
| **`const`** | Runtime (usually) | Variables, member functions | Read-only view of data. Variable cannot be mutated after initialization. | `.rodata` (read-only data segment) |
| **`constexpr`** | Compile-time OR Runtime | Variables, functions | Variable/Function *can* be evaluated at compile-time if conditions allow. | `.rodata` (if not fully inlined/optimized away) |
| **`consteval`** *(C++20)* | **Compile-time ONLY** | Functions | Immediate function. Must be evaluated at compile-time, else compilation fails. | None (only exists in compiler AST) |
| **`constinit`** *(C++20)* | Compile-time initialization | Variables | Forces compile-time static initialization of variable. Variable remains mutable. | `.data` or `.bss` (writable data segments) |

---

## 2. Static Initialization Order Fiasco (SIOF)

In C++, global and static variables defined in a single source file (translation unit) are initialized in the order of their declaration. However, the standard does not guarantee the initialization order of globals defined across **different source files**.

### The SIOF Problem (Buggy Code):
If global `logger` in `logger.cpp` uses global `config` in `config.cpp`, and the compiler decides to initialize `logger` before `config`, the logger accesses uninitialized memory, leading to crashes or undefined states.

```cpp
// File A: config.cpp
#include <string>
std::string globalConfig = "PRODUCTION";

// File B: logger.cpp
#include <iostream>
#include <string>
extern std::string globalConfig;
// BUG: If logger compiles before config, globalConfig is uninitialized garbage!
std::string globalLog = "Running in: " + globalConfig; 
```

### The C++20 Solution: `constinit`
Marking a variable `constinit` forces the compiler to initialize it at compile-time (**constant initialization**) rather than during runtime (**dynamic initialization**), eliminating SIOF.
- Unlike `constexpr`, `constinit` variables **remain mutable** at runtime!

```cpp
// Guaranteed compile-time initialization
constinit int globalMaxConnections = 1000; 

void adjustConnections() {
    globalMaxConnections = 500; // OK: mutable at runtime!
}
```

---

## 3. Compile-Time Evaluation Limits (constexpr Heap Allocations)

Prior to C++20, `constexpr` functions were heavily restricted (no virtual calls, no heap allocations, no try-catch blocks). C++20 relaxed these rules.

### Transient Heap Allocations (C++20)
In C++20, you can allocate heap memory using `new` (or standard containers like `std::vector`) inside a `constexpr` context, provided that **all allocated memory is deleted before the constexpr function evaluation completes**. The heap memory is managed by the compiler's compile-time allocator.

```cpp
#include <vector>
#include <numeric>

// C++20 constexpr function using heap memory (std::vector)
constexpr int computeSum(int limit) {
    std::vector<int> nums(limit); // Transient heap allocation at compile-time!
    std::iota(nums.begin(), nums.end(), 1); // Populate 1 to limit
    
    int sum = 0;
    for (int x : nums) sum += x;
    
    return sum; // std::vector destructor runs, freeing transient heap memory. OK!
}

void demoTransient() {
    constexpr int total = computeSum(100); // Evaluated at compile-time. No runtime allocations!
}
```

---

## 4. `if consteval` (C++23)

C++23 introduced `if consteval` to allow branching based on whether the current execution path is running at compile-time or runtime. This allows optimization (like using optimized hardware assembly instructions at runtime, but falling back to pure C++ code at compile-time).

```cpp
#include <cmath>

constexpr double squareRoot(double x) {
    if consteval {
        // Compile-time path (no access to hardware intrinsics)
        // Use a simple Newton-Raphson approximation
        double val = x;
        for (int i = 0; i < 10; ++i) {
            val = 0.5 * (val + x / val);
        }
        return val;
    } else {
        // Runtime path: use fast CPU instructions
        return std::sqrt(x); 
    }
}
```

---

*<- [[02_Core_Value_Categories|Value Categories]] · [[02_Core_Storage_Duration_and_Linkage|Storage & Linkage ->]]*
