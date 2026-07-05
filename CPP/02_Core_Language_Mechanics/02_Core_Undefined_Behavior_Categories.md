---
tags: [cpp, core, undefined-behavior, strict-aliasing, signed-overflow, null-pointer-check]
links: ["[[02_Core_Index]]", "[[02_Core_Storage_Duration_and_Linkage]]", "[[02_Core_Problems]]"]
---

# Core Language Mechanics -- Undefined Behavior (UB)

*<- [[02_Core_Storage_Duration_and_Linkage|Storage & Linkage]] · [[02_Core_Problems|Problems ->]]*

---

## What is Undefined Behavior?

In C++, **Undefined Behavior (UB)** represents a state where the language specification imposes no constraints on the execution of the program.
- The compiler assumes that **UB never occurs** in a valid program.
- If a execution path contains UB, the compiler optimizes the code under this assumption, which can lead to dead-branch pruning, loop deletion, or security check removals.

---

## 1. Strict Aliasing Rule & Aliasing Exceptions

The **Strict Aliasing Rule** dictates that two pointers of different types cannot point to the same memory location, meaning the compiler assumes their modifications do not overlap.

### Exceptions to the Rule
Pointers of type **`char*`**, **`unsigned char*`**, and **`std::byte*`** are allowed to alias any type. This allows writing raw byte copy routines (like `std::memcpy`).

```cpp
#include <cstring>
#include <bit>
#include <cstdint>

// [Violates Strict Aliasing] - Undefined Behavior
float violateAliasing(float* f, int* i) {
    *f = 2.0f;
    *i = 1073741824; // Reinterpreting float memory as int -> UB!
    return *f;       // Compiler may optimize and return cached 2.0f directly
}

// [Safe C++11 approach: std::memcpy]
float safeMemcpy(float* f, int* i) {
    *f = 2.0f;
    int temp = 1073741824;
    std::memcpy(f, &temp, sizeof(float)); // Safe byte-level copy
    return *f; // Correctly reloads f from memory
}

// [Safe C++20 approach: std::bit_cast]
uint32_t safeBitCast(float f) {
    return std::bit_cast<uint32_t>(f); // Evaluates at compile-time if constexpr
}
```

---

## 2. Signed Integer Overflow Exploits

- **Unsigned integers** wrap modulo $2^W$: $4294967295 + 1 = 0$.
- **Signed integers** overflow triggers **Undefined Behavior**.

### Compiler Optimization Trap:
Because signed overflow is UB, the compiler assumes it cannot happen. Therefore, expressions like `x + 1 > x` are statically optimized to `true` and the check is pruned from the assembly.

```cpp
#include <iostream>

bool checkOverflow(int x) {
    // Compiler assumes x + 1 will never overflow, meaning x + 1 > x is always true.
    // The entire check is optimized away to 'return true;'!
    return x + 1 > x; 
}

void demoOverflow() {
    int maxVal = 2147483647; // INT_MAX
    if (checkOverflow(maxVal)) {
        // Runs even though maxVal + 1 overflows to -2147483648!
        std::cout << "Optimized to true!\n"; 
    }
}
```

### The Safe Fix:
Perform bounds checks using algebra *before* performing the operation:
```cpp
bool safeAddCheck(int x, int y) {
    if (y > 0 && x > 2147483647 - y) return false; // Overflow detected
    if (y < 0 && x < -2147483648 - y) return false; // Underflow detected
    return true;
}
```

---

## 3. Null Pointer Check Pruning (Security Vulnerability)

If a pointer is dereferenced *before* it is checked for null, the compiler assumes the pointer must be non-null (since dereferencing a null pointer is UB, and legal C++ code cannot have UB).
- The compiler will **optimize out** subsequent null checks, which can bypass security locks.

```cpp
struct Session {
    void process() {}
};

void execute(Session* s) {
    s->process(); // Dereferenced here
    
    // ...
    
    if (!s) {
        // BUG: Compiler assumes 's' cannot be null because it was already dereferenced above.
        // This entire block is deleted during compilation!
        return; 
    }
}
```

---

## 4. Unsequenced Modifications (Sequence Points)

An expression is UB if it performs two unsequenced modifications on the same variable, or a modification and a read.

- **Pre-C++17**: Expressions like `i = i++` or `f(i++, i++)` were unsequenced and triggered UB.
- **Post-C++17**: Evaluated arguments are **sequenced** (one argument completes evaluation fully before the next starts), though the relative evaluation order remains unspecified.
- However, expressions like `i = ++i + i++` remain unsequenced and UB in all standards.

---

*<- [[02_Core_Storage_Duration_and_Linkage|Storage & Linkage]] · [[02_Core_Problems|Problems ->]]*
