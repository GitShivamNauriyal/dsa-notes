---
tags: [cpp, core, undefined-behavior, strict-aliasing, signed-overflow, sequence-points]
links: ["[[02_Core_Index]]", "[[02_Core_Storage_Duration_and_Linkage]]", "[[02_Core_Problems]]"]
---

# Core Language Mechanics -- Undefined Behavior (UB)

*<- [[02_Core_Storage_Duration_and_Linkage|Storage & Linkage]] · [[02_Core_Problems|Problems ->]]*

---

## What is Undefined Behavior?

In C++, **Undefined Behavior (UB)** means the language specification places no constraints on what the program can do.
- The compiler generates code under the assumption that **UB never occurs**. 
- If a path contains UB, the compiler is free to optimize away entire check conditions, prune branches, or generate assembly instructions that execute arbitrary code.

---

## 1. Strict Aliasing Rule

The **Strict Aliasing Rule** states that two pointers of different types cannot point to the same memory address, with a few exceptions (like `char*`, `unsigned char*`, and `std::byte*`).
- **Why compilers care**: If two pointers are of different types, the compiler assumes they do not overlap in memory (alias). This allows the compiler to keep values in registers instead of reloading them from memory.
- Dereferencing incompatible types violates strict aliasing and leads to UB.

```cpp
// [Violates Strict Aliasing]
float violateAliasing(float* f, int* i) {
    *f = 2.0f;
    *i = 10; // UB: i aliases the memory of float f
    return *f; // Compiler might optimize this to return 2.0f directly without reloading memory
}

// [Safe: C++20 std::bit_cast]
#include <bit>
#include <cstdint>

uint32_t inspectBits(float f) {
    // Safely copy bits without pointer casting
    return std::bit_cast<uint32_t>(f);
}
```

---

## 2. Signed Integer Overflow vs Unsigned Wrapping

- **Unsigned integers** have defined modulo wrapping behavior: $2^{32} - 1 + 1 = 0$.
- **Signed integers** overflow is **Undefined Behavior**.
- **Compiler exploitation**: Because signed overflow is UB, the compiler assumes it cannot happen. Therefore, it will optimize out safety checks!

```cpp
// The compiler assumes (x + 1 > x) is ALWAYS true because overflow is impossible in legal C++.
// The safety check is optimized out completely!
bool checkOverflow(int x) {
    return x + 1 > x; // Compiler optimizes this to return 'true' directly!
}
```

---

## 3. Order of Evaluation & Sequence Points

An expression is UB if it performs two unsequenced modifications on the same variable, or a modification and a read.

### Pre-C++17 vs Post-C++17 Rules
- **Pre-C++17**: Expressions like `i = i++` or `f(i++, i++)` were completely unsequenced and triggered UB.
- **Post-C++17**: The order of evaluation of function arguments remains unspecified, but they are guaranteed to be **sequenced** (one argument completes evaluation fully before the other starts). Shift operators `a << b` evaluate `a` before `b`.
- However, expressions like `i = ++i + i++` remain unsequenced and UB.

```cpp
#include <iostream>

void printTwo(int a, int b) {
    std::cout << a << " " << b << "\n";
}

void demoSequence() {
    int i = 0;
    // Post-C++17: Evaluated sequentially, but which one goes first is unspecified.
    // Pre-C++17: UB!
    printTwo(i++, i++); 
}
```

---

## 4. Dangling References

Accessing an object whose storage duration has ended.

```cpp
const int& getDangling() {
    int localVal = 42;
    return localVal; // UB: localVal is deallocated when stack frame exits
}

void process() {
    const int& ref = getDangling();
    std::cout << ref << "\n"; // UB: dereferencing stack garbage!
}
```

---

*<- [[02_Core_Storage_Duration_and_Linkage|Storage & Linkage]] · [[02_Core_Problems|Problems ->]]*
