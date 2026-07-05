---
tags: [cpp, templates, metaprogramming, compile-time-recursion, type-selection]
links: ["[[04_Tmpl_Index]]", "[[04_Tmpl_Concepts_Cpp20]]", "[[04_Tmpl_Problems]]"]
---

# Templates & Generics -- Template Metaprogramming (TMP)

*<- [[04_Tmpl_Concepts_Cpp20|Concepts (C++20)]] · [[04_Tmpl_Problems|Problems ->]]*

---

## 1. What is Template Metaprogramming?

**Template Metaprogramming (TMP)** is a technique where the compiler executes code at compile-time to compute values or generate customized classes. 
- In TMP, templates act as a functional, Turing-complete sub-language.
- Loops are replaced with **recursion**, and variables are replaced with **nested types or constants**.

---

## 2. Compile-Time Recursion

To compute a value at compile-time, we use recursive template instantiations. The recursion ends at a specialized base case.

### Example: Compile-Time Factorial

```cpp
#include <iostream>

// Primary template: recursive step
template <unsigned int N>
struct Factorial {
    static constexpr unsigned long long value = N * Factorial<N - 1>::value;
};

// Template specialization: base case
template <>
struct Factorial<0> {
    static constexpr unsigned long long value = 1;
};

void demoFactorial() {
    // Value computed entirely by the compiler!
    unsigned long long val = Factorial<5>::value; // compiles as if you wrote: val = 120;
}
```

### Instantiation Chain Diagram
When the compiler resolves `Factorial<3>::value`, it generates the following instantiation chain:

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

## 3. Type Selection: `std::conditional`

In addition to values, TMP allows manipulating **types** at compile-time.
- **`std::conditional<Condition, Type1, Type2>::type`** acts as a compile-time `if-else` block for choosing types.
- If `Condition` is `true`, it resolves to `Type1`. Otherwise, it resolves to `Type2`.

```cpp
#include <type_traits>

// Choose appropriate integer representation based on size requirement
template <unsigned int Bytes>
struct IntSelector {
    using type = typename std::conditional<Bytes <= 1, char,
                 typename std::conditional<Bytes <= 2, short,
                 typename std::conditional<Bytes <= 4, int, long long>::type
                 >::type
                 >::type;
};

void demoSelector() {
    IntSelector<2>::type val1; // Resolves to 'short'
    IntSelector<8>::type val2; // Resolves to 'long long'
}
```

---

*<- [[04_Tmpl_Concepts_Cpp20|Concepts (C++20)]] · [[04_Tmpl_Problems|Problems ->]]*
