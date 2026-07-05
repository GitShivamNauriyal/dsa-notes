---
tags: [cpp, stl, optional, variant, tuple, structured-bindings, monadic-operations, Cpp23]
links: ["[[05_STL_Index]]", "[[05_STL_Containers_and_Iterator_Invalidation]]", "[[05_STL_Ranges_and_Views]]"]
---

# STL & Modern Features -- Optional, Variant, & Tuple

*<- [[05_STL_Containers_and_Iterator_Invalidation|Containers & Invalidation]] · [[05_STL_Ranges_and_Views|Ranges & Views ->]]*

---

## 1. `std::optional` (C++17) & C++23 Monadic Operations

`std::optional<T>` represents a value that may or may not exist.

### Size Overhead:
- **Memory Footprint**: `sizeof(std::optional<T>)` is equal to `sizeof(T) + sizeof(bool)` plus alignment padding.
- For example, `sizeof(std::optional<int>)` occupies **8 bytes** on 64-bit systems (4 bytes for `int`, 1 byte for the active flag, aligned to the 4-byte boundary).

### C++23 Monadic Operations
C++23 added Rust-like monadic interfaces to `std::optional` to chain transformations cleanly without nested `if (opt.has_value())` blocks.

- **`.and_then(F)`**: If value exists, returns `F(*opt)`. F must return a `std::optional`.
- **`.transform(F)`**: If value exists, applies `F(*opt)` and wraps the result in a new `std::optional`.
- **`.or_else(F)`**: If empty, runs `F()` (which must return a `std::optional`).

```cpp
#include <optional>
#include <string>
#include <iostream>

std::optional<std::string> fetchUser(int id) {
    if (id == 4) return "User4";
    return std::nullopt;
}

std::optional<std::string> makeUppercase(std::string name) {
    for (char& c : name) c = std::toupper(c);
    return name;
}

void demoMonadic() {
    // Monadic chaining (C++23)
    auto result = fetchUser(4)
                .and_then(makeUppercase)
                .value_or("ANONYMOUS");
                
    std::cout << result << "\n"; // Outputs: USER4
}
```

---

## 2. `std::variant` & `std::visit` (C++17 Type-Safe Union)

`std::variant<Types...>` represents a type-safe union. It holds exactly one of its active types.

### Memory Overhead:
- **No Dynamic Allocation**: Unlike polymorphic base class structures, `std::variant` stores its contents **entirely on the stack**. No `new` or `malloc` calls are made.
- **Size**: Equal to `sizeof(largest_type)` + size of an internal index indicator variable + alignment padding.

### Visited Overload Pattern
Using the helper struct `overloaded` (which uses C++17 template pack expansion on base class constructors) allows visiting different types inside a variant using inline lambdas:

```cpp
#include <variant>
#include <iostream>
#include <string>

// Variadic template struct inherits from all lambdas and exposes their call operator
template<class... Ts> struct overloaded : Ts... { using Ts::operator()...; };
template<class... Ts> overloaded(Ts...) -> overloaded<Ts...>;

void demoVariant() {
    std::variant<int, double, std::string> v = 3.14;

    std::visit(overloaded {
        [](int x) { std::cout << "int: " << x << "\n"; },
        [](double x) { std::cout << "double: " << x << "\n"; },
        [](const std::string& x) { std::cout << "string: " << x << "\n"; }
    }, v); // Resolves to double output at runtime (based on active index)
}
```

---

## 3. `std::tuple` & Structured Bindings (C++17)

A `std::tuple` stores a fixed-size collection of heterogeneous values.

### Structured Bindings Under the Hood
When you write:
```cpp
std::tuple<int, double> record = {1, 2.5};
auto [id, score] = record;
```

The compiler translates it by introducing a hidden compiler-generated variable:
```cpp
auto&& __temp = record;
// Binds references or values to the tuple elements using std::get
std::tuple_element_t<0, std::tuple<int, double>>& id = std::get<0>(__temp);
std::tuple_element_t<1, std::tuple<int, double>>& score = std::get<1>(__temp);
```
- There is **zero runtime overhead**. The mapping is resolved entirely at compile-time.

---

*<- [[05_STL_Containers_and_Iterator_Invalidation|Containers & Invalidation]] · [[05_STL_Ranges_and_Views|Ranges & Views ->]]*
