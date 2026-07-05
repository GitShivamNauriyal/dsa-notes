---
tags: [cpp, templates, sfinae, type-traits, enable-if, void-t, compile-check]
links: ["[[04_Tmpl_Index]]", "[[04_Tmpl_Variadic_Templates]]", "[[04_Tmpl_Concepts_Cpp20]]"]
---

# Templates & Generics -- SFINAE & Type Traits

*<- [[04_Tmpl_Variadic_Templates|Variadic Templates]] · [[04_Tmpl_Concepts_Cpp20|Concepts (C++20) ->]]*

---

## 1. What is SFINAE?

**SFINAE** (**Substitution Failure Is Not An Error**) is a core rule of C++ overload resolution:
- When matching a function call, the compiler substitutes the template arguments into every candidate template overload.
- If a substitution results in an invalid type or expression (e.g. attempting to dereference a non-pointer, or access a non-existent nested type), **compilation does not fail**.
- The compiler simply discards that specific candidate from the overload candidate set and proceeds to check the others.
- A compilation error occurs **only** if no matching overloads are left in the set.

---

## 2. Type Traits (`<type_traits>`)

Type traits are compile-time templates that allow you to query properties of types.

### Primary Helpers
- **`std::is_integral_v<T>`**: True if `T` is an integral type (e.g. `int`, `char`, `bool`).
- **`std::is_floating_point_v<T>`**: True if `T` is `float` or `double`.
- **`std::is_pointer_v<T>`**: True if `T` is a raw pointer.
- **`std::decay_t<T>`**: Removes `const`, `volatile`, and reference qualifiers, and decays arrays and functions into pointers.

```cpp
#include <type_traits>

template <typename T>
void inspectType() {
    // std::decay strips const and reference qualifiers to inspect raw type
    using CleanType = typename std::decay<T>::type; 
    
    if (std::is_integral<CleanType>::value) {
        // ...
    }
}
```

---

## 3. `std::enable_if` Overload Filtering

`std::enable_if` is a template structure that conditionally defines a nested type `type` if its condition parameter evaluates to `true`.
- If the condition is `false`, `std::enable_if` lacks a nested `type`. Attempting to access it triggers SFINAE, discarding the overload.

### Code Implementation (Enable If Overloads)

```cpp
#include <type_traits>
#include <iostream>

// Overload A: Activated for integral types
template <typename T>
typename std::enable_if<std::is_integral<T>::value, void>::type
process(T val) {
    std::cout << "Integer value: " << val << "\n";
}

// Overload B: Activated for floating-point types
template <typename T>
typename std::enable_if<std::is_floating_point<T>::value, void>::type
process(T val) {
    std::cout << "Float value: " << val << "\n";
}
```

---

## 4. C++17 `std::void_t` (The Modern SFINAE Pattern)

Before C++20 Concepts, checking if a type supported a specific operation (e.g., whether a class has a member function `serialize()`, or supports addition) required complex boilerplate. C++17 introduced **`std::void_t`** to simplify this.

- **`std::void_t<Ts...>`**: A template alias that always maps any pack of types to `void`.
- **How it works**: If any of the types in the pack are invalid expressions (due to SFINAE), the template substitution fails.

### Example: Checking for Member Function `serialize`
Let's build a compile-time trait to check if a class contains a method `void serialize()`.

```cpp
#include <type_traits>
#include <iostream>

// 1. Primary template (default case: does not have serialize)
template <typename T, typename = void>
struct has_serialize : std::false_type {};

// 2. Specialization using void_t
// If decltype(std::declval<T>().serialize()) is valid, this specialization matches!
template <typename T>
struct has_serialize<T, std::void_t<decltype(std::declval<T>().serialize())>> : std::true_type {};

// Test structures
struct XMLPrinter {
    void serialize() {}
};

struct EmptyStruct {};

void demoVoidT() {
    std::cout << has_serialize<XMLPrinter>::value << "\n";  // Outputs 1 (true)
    std::cout << has_serialize<EmptyStruct>::value << "\n"; // Outputs 0 (false)
}
```

---

*<- [[04_Tmpl_Variadic_Templates|Variadic Templates]] · [[04_Tmpl_Concepts_Cpp20|Concepts (C++20) ->]]*
