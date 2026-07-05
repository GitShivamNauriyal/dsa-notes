---
tags: [cpp, templates, sfinae, type-traits, enable-if]
links: ["[[04_Tmpl_Index]]", "[[04_Tmpl_Variadic_Templates]]", "[[04_Tmpl_Concepts_Cpp20]]"]
---

# Templates & Generics -- SFINAE & Type Traits

*<- [[04_Tmpl_Variadic_Templates|Variadic Templates]] · [[04_Tmpl_Concepts_Cpp20|Concepts (C++20) ->]]*

---

## 1. What is SFINAE?

**SFINAE** stands for **Substitution Failure Is Not An Error**.
- When the compiler resolves a template function call, it examines all overloaded candidates.
- It attempts to substitute the template parameters with the argument types.
- If a substitution results in an invalid type or expression, the compiler **does not generate a compilation error**. Instead, it silently discards that candidate from the overload resolution set.
- A compilation error only occurs if no valid overloads remain.

---

## 2. Type Traits (`<type_traits>`)

Type traits are template structures that query or modify properties of types at compile-time:
- **`std::is_integral<T>::value`**: True if `T` is `int`, `char`, `long`, etc.
- **`std::is_floating_point<T>::value`**: True if `T` is `float` or `double`.
- **`std::is_pointer<T>::value`**: True if `T` is a raw pointer.

In C++14, helper variables ending with `_v` were introduced to simplify syntax:
`std::is_integral_v<T>` is equivalent to `std::is_integral<T>::value`.

---

## 3. Conditionally Enabling Overloads: `std::enable_if`

`std::enable_if<Condition, Type>::type` compiles only if `Condition` is `true`. If `Condition` is `false`, it fails substitution, triggering SFINAE.
- **C++14 Shorthand**: `std::enable_if_t<Condition, Type>`.

### Example: Different Algorithms for Integers vs Floats

```cpp
#include <type_traits>
#include <iostream>

// Overload 1: Enabled only for integral types
template <typename T>
typename std::enable_if<std::is_integral<T>::value, void>::type
processNumber(T val) {
    std::cout << "Integer processing: " << val << "\n";
}

// Overload 2: Enabled only for floating-point types (using C++14 _t / _v helpers)
template <typename T, typename = std::enable_if_t<std::is_floating_point_v<T>>>
void processNumber(T val) {
    std::cout << "Floating-point processing: " << val << "\n";
}

void demoSFINAE() {
    processNumber(10);   // Calls Overload 1 (T = int)
    processNumber(3.14); // Calls Overload 2 (T = double)
    // processNumber("Hello"); // COMPILER ERROR: No matching overload! (both failed substitution)
}
```

---

*<- [[04_Tmpl_Variadic_Templates|Variadic Templates]] · [[04_Tmpl_Concepts_Cpp20|Concepts (C++20) ->]]*
