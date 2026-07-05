---
tags: [cpp, core, problems, exercises]
links: ["[[02_Core_Index]]", "[[02_Core_Undefined_Behavior_Categories]]", "[[02_Core_Solutions]]"]
---

# Core Language Mechanics -- Problems & Exercises

*<- [[02_Core_Undefined_Behavior_Categories|Undefined Behavior]] · [[02_Core_Solutions|Solutions ->]]*

---

## Tier 1 -- Concept Checks

### Question 1.1: Value Categories
Given the declarations:
```cpp
int x = 10;
int& lref = x;
int&& rref = 20;
```
Identify the value category (lvalue, prvalue, xvalue) of each of the following expressions:
1. `x`
2. `lref`
3. `rref`
4. `std::move(x)`
5. `42`
6. `x + 5`

---

### Question 1.2: Linkage and Storage Duration
Determine the linkage (external, internal, none) and storage duration (static, automatic, thread, dynamic) of the following variables:
1. A global variable declared in a namespace: `namespace App { int count = 0; }`
2. A variable declared inside a function with the `static` keyword: `static int counter = 0;`
3. A global variable declared with `static`: `static int internalVal = 10;`
4. A local parameter inside a function: `void foo(int x)`

---

## Tier 2 -- Implementation

### Question 2.1: Constexpr Factorial
Write a `constexpr` function `constexpr long long factorial(int n)` that computes the factorial of $n$. Show how to:
1. Force it to be evaluated at compile-time by assigning it to a constant.
2. Call it at runtime using variables.

---

## Tier 3 -- Interview-Level

### Question 3.1: Strict Aliasing Violation Debugging
A developer wrote the following code to inspect the raw IEEE-754 representation of a float:
```cpp
float f = 5.5f;
int* pi = (int*)&f; // C-style cast
int bits = *pi;    // Dereferencing
```
1. Explain why this violates the strict aliasing rule and leads to Undefined Behavior.
2. Show how to rewrite this safely in C++11 (using `std::memcpy`), and in C++20 (using `std::bit_cast`).

---

### Question 3.2: Compile-Time Verification of Array Size
Design a macro or constexpr function in C++11 that checks if a static C-style array has a specific size at compile-time. If the size does not match, trigger a compile-time error.

---

## Tier 4 -- Systems / Placement-Hard

### Question 4.1: Custom `bit_cast` for C++11 Floor
C++20 introduced `std::bit_cast`. Write a custom utility `template <typename Dest, typename Source> Dest custom_bit_cast(const Source& src)` that:
- Runs on a C++11 floor compiler.
- Reinterprets the bits of `src` into type `Dest`.
- Prevents compilation if `sizeof(Dest) != sizeof(Source)`.
- Restricts type copyability properties safely to prevent dereferencing UB.

---

*<- [[02_Core_Undefined_Behavior_Categories|Undefined Behavior]] · [[02_Core_Solutions|Solutions ->]]*
