---
tags: [cpp, stl, optional, variant, tuple, structured-bindings]
links: ["[[05_STL_Index]]", "[[05_STL_Containers_and_Iterator_Invalidation]]", "[[05_STL_Ranges_and_Views]]"]
---

# STL & Modern Features -- Optional, Variant, & Tuple

*<- [[05_STL_Containers_and_Iterator_Invalidation|Containers & Invalidation]] · [[05_STL_Ranges_and_Views|Ranges & Views ->]]*

---

## 1. `std::optional` (C++17)

`std::optional<T>` manages an optional contained value, which may or may not be present. It eliminates the need for error-prone sentinel values (like returning `-1` or `nullptr`).

```cpp
#include <optional>
#include <string>

std::optional<std::string> getUserName(int id) {
    if (id == 42) return "Alice";
    return std::nullopt; // represents empty state
}

void demoOptional() {
    auto nameOpt = getUserName(42);
    
    if (nameOpt.has_value()) {
        std::string name = nameOpt.value(); // or *nameOpt
    }
    
    // Fallback default value if empty
    std::string finalName = nameOpt.value_or("Guest");
}
```

---

## 2. `std::variant` & `std::visit` (C++17 type-safe union)

`std::variant<Types...>` is a type-safe alternative to C-style unions. It holds exactly one of its specified types at any given time.

### Visited Overload Pattern
To process the active type inside a variant, we use **`std::visit`** along with a helper class template that overloads `operator()` for different types:

```cpp
#include <variant>
#include <iostream>
#include <string>

// Helper structure to overload lambda operators
template<class... Ts> struct overloaded : Ts... { using Ts::operator()...; };
template<class... Ts> overloaded(Ts...) -> overloaded<Ts...>;

void demoVariant() {
    // Can hold either int, double, or std::string
    std::variant<int, double, std::string> var = "Hello";

    // Visit using overloaded lambda structures
    std::visit(overloaded {
        [](int arg) { std::cout << "int: " << arg << '\n'; },
        [](double arg) { std::cout << "double: " << arg << '\n'; },
        [](const std::string& arg) { std::cout << "string: " << arg << '\n'; }
    }, var);
}
```

---

## 3. `std::tuple` & Structured Bindings (C++17)

A `std::tuple` is a generalization of `std::pair` that holds an arbitrary number of heterogeneous values.
C++17 **Structured Bindings** allow unpacking tuples, pairs, or structs into individual variables easily.

```cpp
#include <tuple>
#include <string>

std::tuple<int, std::string, double> getEmployeeRecord() {
    return {101, "Bob", 75000.0};
}

void demoTuple() {
    // Unpack tuple into local variables using structured bindings
    auto [id, name, salary] = getEmployeeRecord();
}
```

---

*<- [[05_STL_Containers_and_Iterator_Invalidation|Containers & Invalidation]] · [[05_STL_Ranges_and_Views|Ranges & Views ->]]*
