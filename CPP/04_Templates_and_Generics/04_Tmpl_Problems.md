---
tags: [cpp, templates, problems, exercises]
links: ["[[04_Tmpl_Index]]", "[[04_Tmpl_Metaprogramming_Intro]]", "[[04_Tmpl_Solutions]]"]
---

# Templates & Generics -- Problems & Exercises

*<- [[04_Tmpl_Metaprogramming_Intro|Metaprogramming Intro]] · [[04_Tmpl_Solutions|Solutions ->]]*

---

## Tier 1 -- Concept Checks

### Question 1.1: Function Template Specialization
Predict if the following code compiles. If it compiles, what does it print? If it fails, explain the exact C++ template rules violated:
```cpp
#include <iostream>

template <typename T, typename U>
void printVal(T a, U b) {
    std::cout << "Primary\n";
}

// Case A
template <typename T>
void printVal<T, int>(T a, int b) {
    std::cout << "Specialized A\n";
}

int main() {
    printVal(5.5, 10);
    return 0;
}
```

---

### Question 1.2: Variadic Template Instantiation count
Given the template:
```cpp
template <typename... Args>
int getArgsCount(Args... args) {
    return sizeof...(args);
}
```
If we call `getArgsCount(1, 2.5, "Hello")`:
1. What does it return?
2. How many distinct function instances does the compiler generate for the entire binary?

---

## Tier 2 -- Implementation

### Question 2.1: Print Pack Fold Expression
Implement a function template `template <typename... Args> void printPack(Args... args)` using C++17 **Fold Expressions** that:
- Prints all arguments to `std::cout` separated by a comma and a space.
- Does not print a trailing comma after the last argument.
- E.g., `printPack(1, "two", 3.0)` should output: `1, two, 3.0`.

---

## Tier 3 -- Interview-Level

### Question 3.1: Re-implementing Type Traits
In interview or systems settings, you might be asked to implement basic library features to prove you understand their mechanisms.
Write a custom template `struct is_pointer_type<T>` from scratch **without using `<type_traits>`** that:
- Contains a member static constant `value` set to `true` if `T` is a pointer, and `false` otherwise.
- Supports pointer-to-const types (e.g. `const int*` should evaluate to `true`).

---

### Question 3.2: SFINAE Member Checker
Write a template struct `has_serialize_member<T>` using C++11 SFINAE that:
- Contains `value = true` if class `T` contains a member function `void serialize()`, and `false` otherwise.
- Provides test structs with and without this function to verify compile-time resolution.

---

## Tier 4 -- Systems / Placement-Hard

### Question 4.1: Compile-Time Typelist and Length Checker
In advanced template libraries, lists of types (**typelists**) are used to configure objects (e.g. databases columns, message fields).
1. Design a compile-time `TypeList` structure that holds a list of types.
2. Implement a meta-function `TypeListLength<TL>::value` that computes the number of types in the list at compile-time.
3. Implement a meta-function `TypeAt<TL, Index>::type` that retrieves the type at a specific 0-based index. If index is out of bounds, fail compilation.

---

*<- [[04_Tmpl_Metaprogramming_Intro|Metaprogramming Intro]] · [[04_Tmpl_Solutions|Solutions ->]]*
