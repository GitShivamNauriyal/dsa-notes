---
tags: [cpp, templates, concepts, constraints, Cpp20, subsumption]
links: ["[[04_Tmpl_Index]]", "[[04_Tmpl_SFINAE_and_Type_Traits]]", "[[04_Tmpl_Metaprogramming_Intro]]"]
---

# Templates & Generics -- Concepts (C++20)

*<- [[04_Tmpl_SFINAE_and_Type_Traits|SFINAE & Type Traits]] · [[04_Tmpl_Metaprogramming_Intro|Metaprogramming Intro ->]]*

---

## 1. Concepts: Modern Constraint Programming

In C++20, **Concepts** are compile-time predicates that restrict template parameter types. They replace the complex template substitutions of SFINAE.

### Key Advantages:
1. **Clear Compiler Errors**: Instead of page-long tracebacks, the compiler prints readable messages like `type std::string does not satisfy concept std::integral`.
2. **Readability**: Constraints are declared inline without verbose helper wrappers.
3. **Compilation Speed**: The compiler checks boolean concepts directly instead of executing dummy template instantiations.

---

## 2. Defining Concepts & the `requires` Expression

Concepts are defined using the `concept` keyword. A `requires` expression contains a body of constraints that must compile successfully.

```cpp
#include <concepts>
#include <type_traits>

template <typename T>
concept Numeric = std::is_integral_v<T> || std::is_floating_point_v<T>;
```

### The Four Types of `requires` Requirements:
1. **Simple Requirement**: An expression that must compile.
2. **Type Requirement**: A nested type that must exist (`typename T::value_type`).
3. **Compound Requirement**: An expression that must compile and whose return type must satisfy a concept (`{ expr } noexcept -> concept;`).
4. **Nested Requirement**: A compile-time boolean expression that must evaluate to `true` (`requires sizeof(T) >= 4;`).

```cpp
#include <concepts>

template <typename T>
concept PrintableContainer = requires(T container, typename T::value_type val) {
    // 1. Type requirement: container must define a nested value_type
    typename T::value_type;
    
    // 2. Simple requirement: must have begin() and end()
    container.begin();
    container.end();
    
    // 3. Compound requirement: size() must compile and return a type matching std::integral
    { container.size() } noexcept -> std::integral;
    
    // 4. Nested requirement: elements must be at least 4 bytes in size
    requires sizeof(val) >= 4;
};
```

---

## 3. Four Ways to Apply Concepts

You can enforce concepts in template definitions using four different syntaxes depending on readability preferences:

### 1. Template Parameter Syntax (Inline)
```cpp
template <Numeric T>
T add(T a, T b) { return a + b; }
```

### 2. Requires Clause (Before signature)
```cpp
template <typename T>
requires Numeric<T>
T add(T a, T b) { return a + b; }
```

### 3. Trailing Requires Clause (After signature)
```cpp
template <typename T>
T add(T a, T b) requires Numeric<T> { return a + b; }
```

### 4. Constrained Auto (Syntactic Sugar)
```cpp
Numeric auto add(Numeric auto a, Numeric auto b) {
    return a + b;
}
```

---

## 4. Concept Subsumption (Constraint Ordering)

If multiple template overloads are valid for a call, the compiler chooses the **most constrained** overload. A concept $C_1$ **subsumes** (is stricter than) $C_2$ if $C_1$ requires everything $C_2$ requires, plus additional constraints.
- This eliminates the need for SFINAE tag-dispatching.

```cpp
#include <concepts>
#include <iostream>

template <typename T>
concept Bidirectional = requires(T x) {
    { --x } -> std::same_as<T&>;
};

template <typename T>
concept RandomAccess = Bidirectional<T> && requires(T x) {
    { x + 1 } -> std::same_as<T>;
};

// Overload A: Binds to any Bidirectional type
template <Bidirectional T>
void advance(T& it) {
    std::cout << "Bidirectional linear stepping\n";
}

// Overload B: Binds to RandomAccess types (subsumes Overload A)
template <RandomAccess T>
void advance(T& it) {
    std::cout << "RandomAccess constant jump\n";
}

void demoSubsumption() {
    int* ptr = nullptr; // Pointer is a RandomAccess iterator
    advance(ptr);       // Resolves to Overload B (stricter constraint chosen!)
}
```

---

*<- [[04_Tmpl_SFINAE_and_Type_Traits|SFINAE & Type Traits]] · [[04_Tmpl_Metaprogramming_Intro|Metaprogramming Intro ->]]*
