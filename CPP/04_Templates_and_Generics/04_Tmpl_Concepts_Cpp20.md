---
tags: [cpp, templates, concepts, constraints, Cpp20]
links: ["[[04_Tmpl_Index]]", "[[04_Tmpl_SFINAE_and_Type_Traits]]", "[[04_Tmpl_Metaprogramming_Intro]]"]
---

# Templates & Generics -- Concepts (C++20)

*<- [[04_Tmpl_SFINAE_and_Type_Traits|SFINAE & Type Traits]] · [[04_Tmpl_Metaprogramming_Intro|Metaprogramming Intro ->]]*

---

## 1. What are Concepts?

**Concepts** (C++20) provide a clean way to restrict template parameters by defining compile-time constraints. They replace the verbose and unreadable SFINAE templates.

### Why Concepts are a Game Changer:
1. **Readable Compiler Errors**: Instead of pages of template tracebacks, the compiler states exactly which constraint failed (e.g. `type 'std::string' does not satisfy concept 'std::integral'`).
2. **Simplified Code**: Removes SFINAE wrappers (`std::enable_if_t`) completely.
3. **Faster Compilations**: Compiler evaluates constraints directly rather than trying multiple template substitution branches.

---

## 2. Defining and Constraining Concepts

You define a concept using the `concept` keyword, which evaluates a compile-time boolean expression:

```cpp
#include <type_traits>
#include <concepts>

// Define a simple concept
template <typename T>
concept Numeric = std::is_integral_v<T> || std::is_floating_point_v<T>;
```

### The `requires` Expression
`requires` expressions verify whether specific syntactic operations (like calling a method, dereferencing, adding) are valid for a type:

```cpp
template <typename T>
concept Hashable = requires(T a) {
    // Requires that std::hash<T>{}(a) compiles and returns a type convertible to std::size_t
    { std::hash<T>{}(a) } -> std::convertible_to<std::size_t>;
};
```

---

## 3. Four Ways to Apply Concepts

Here is how you can use the `Numeric` concept to constrain a template:

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
// Clearest syntax: no template brackets!
Numeric auto add(Numeric auto a, Numeric auto b) {
    return a + b;
}
```

---

*<- [[04_Tmpl_SFINAE_and_Type_Traits|SFINAE & Type Traits]] · [[04_Tmpl_Metaprogramming_Intro|Metaprogramming Intro ->]]*
