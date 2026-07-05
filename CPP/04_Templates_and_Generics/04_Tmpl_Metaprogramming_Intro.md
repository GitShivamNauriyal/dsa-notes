---
tags: [cpp, templates, metaprogramming, compile-time-recursion, constexpr, consteval]
links: ["[[04_Tmpl_Index]]", "[[04_Tmpl_Concepts_Cpp20]]", "[[04_Tmpl_Problems]]"]
---

# Templates & Generics -- Template Metaprogramming (TMP)

*<- [[04_Tmpl_Concepts_Cpp20|Concepts (C++20)]] · [[04_Tmpl_Problems|Problems ->]]*

---

## 1. What is Template Metaprogramming?

**Template Metaprogramming (TMP)** is a technique where the compiler executes code at compile-time to compute values or generate customized class definitions.
- In TMP, the compiler's template instantiation engine acts as a functional, Turing-complete compiler-sub-language.
- Iterative loops are replaced with **recursive template instantiations**.
- Variables are replaced with **nested static constants or typedefs**.

---

## 2. Value Metaprogramming (Compile-Time Recursion)

Before C++11, value computations at compile-time were written using recursive structs and template specialization.

```cpp
#include <iostream>

// Primary template: recursive step
template <unsigned int N>
struct Factorial {
    static constexpr unsigned long long value = N * Factorial<N - 1>::value;
};

// Template specialization: base case (terminates recursion)
template <>
struct Factorial<0> {
    static constexpr unsigned long long value = 1;
};

void demoFactorial() {
    // Computed entirely by the compiler during parsing!
    unsigned long long val = Factorial<5>::value; // compiles as if you wrote: val = 120;
}
```

### Instantiation Chain Diagram
When the compiler encounters `Factorial<3>::value`, it generates the following instantiation chain:

```
 Factorial<3>::value ──► 3 * Factorial<2>::value
                                  │
                                  ▼
                            2 * Factorial<1>::value
                                      │
                                      ▼
                                1 * Factorial<0>::value (Base Case specialized = 1)
```

---

## 3. Type Metaprogramming (Type Selection)

While value computations are now mostly replaced by `constexpr` (see Section 5), **type manipulation** remains the exclusive domain of templates.

- **`std::conditional<Condition, T, F>::type`** acts as a compile-time conditional type selector.
- If `Condition` is `true`, it resolves to type `T`. Otherwise, it resolves to `F`.

```cpp
#include <type_traits>

// Select the smallest integer type capable of holding N bytes
template <std::size_t Bytes>
struct FitInt {
    using type = typename std::conditional_t<Bytes <= 1, char,
                 typename std::conditional_t<Bytes <= 2, short,
                 typename std::conditional_t<Bytes <= 4, int, long long>
                 >>;
};

void demoSelection() {
    FitInt<2>::type a; // Declared as short
    FitInt<8>::type b; // Declared as long long
}
```

---

## 4. Compile-Time Traversal: Trait Dispatches

We can combine parameter packs and templates to inspect and print types at compile-time.

```cpp
#include <iostream>
#include <typeinfo>

// Traversal base case
void printTypes() {
    std::cout << "\n";
}

// Recursive Type Traversal
template <typename Head, typename... Tail>
void printTypes() {
    std::cout << typeid(Head).name() << " ";
    printTypes<Tail...>(); // Recurse on remaining pack types
}

void demoTypeTraversal() {
    printTypes<int, double, char>(); // Prints type names
}
```

---

## 5. Modern Evolution: Template Metaprogramming vs `constexpr` / `consteval`

Traditional template recursion (`struct Factorial`) was the only compile-time computation tool prior to C++11. Modern C++ has vastly simplified value-metaprogramming.

### The Shift to Imperative Compile-Time Code:
- **`constexpr`** (C++11): Allows writing functions that can run at compile-time if their arguments are constants, using standard imperative syntax (loops, conditions) instead of recursive template hacks.
- **`consteval`** (C++20): Declares an **immediate function** that *must* evaluate at compile-time. If it cannot, the compiler throws an error.

```cpp
// Modern C++20: guaranteed compile-time calculation using normal loops!
consteval unsigned long long factorialConsteval(unsigned int n) {
    unsigned long long res = 1;
    for (unsigned int i = 1; i <= n; ++i) {
        res *= i;
    }
    return res;
}

void demoConsteval() {
    constexpr auto val = factorialConsteval(5); // Evaluated at compile-time
}
```

### When to Use Templates vs `constexpr` / `consteval`:
- Use **`constexpr`/`consteval`** for **value calculations** (math, hashing, parsing strings).
- Use **templates** for **type-level computations** (type traits, typelists, policy-based designs, template class generation).

---

*<- [[04_Tmpl_Concepts_Cpp20|Concepts (C++20)]] · [[04_Tmpl_Problems|Problems ->]]*
