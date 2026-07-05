---
tags: [cpp, templates, variadic-templates, parameter-packs, fold-expressions, pack-expansion]
links: ["[[04_Tmpl_Index]]", "[[04_Tmpl_Specialization]]", "[[04_Tmpl_SFINAE_and_Type_Traits]]"]
---

# Templates & Generics -- Variadic Templates

*<- [[04_Tmpl_Specialization|Template Specialization]] · [[04_Tmpl_SFINAE_and_Type_Traits|SFINAE & Type Traits ->]]*

---

## 1. Parameter Packs & Size Query

A **variadic template** (introduced in C++11) is a template that accepts an arbitrary number of arguments of any type.
- **Template Parameter Pack**: `typename... Args` declares a pack of types.
- **Function Parameter Pack**: `Args... args` declares a pack of values.
- **`sizeof...` Operator**: Returns the number of elements inside a parameter pack as a compile-time constant expression.

```cpp
#include <cstddef>

template <typename... Args>
std::size_t getPackSize(Args... args) {
    // Evaluates at compile-time
    return sizeof...(args); 
}
```

---

## 2. Advanced Pack Expansion Contexts

Parameter packs are expanded by writing `...` to the right of the pattern. You can expand packs in several different C++ syntactic contexts:

### A. Template Argument Lists & Base Class Inheritances
You can inherit from a variable number of classes and initialize them simultaneously:

```cpp
#include <iostream>

struct Reader { void read() { std::cout << "Read\n"; } };
struct Writer { void write() { std::cout << "Write\n"; } };

// Inherit from multiple classes passed in template pack
template <typename... Interfaces>
class Device : public Interfaces... {
public:
    // Expand inside constructor initializer list
    Device(Interfaces... inputs) : Interfaces(inputs)... {}
};

void demoDevice() {
    Reader r;
    Writer w;
    Device<Reader, Writer> dev(r, w);
    dev.read();
    dev.write();
}
```

### B. Array Initializer Lists
Expand a pack directly into an array declaration:

```cpp
template <typename... Args>
void populateArray(Args... args) {
    // Expands to: int arr[] = { args1, args2, args3 };
    int arr[] = { args... }; 
}
```

---

## 3. Fold Expressions (C++17)

Prior to C++17, operating on a pack required writing a recursive helper function with a base-case overload. C++17 introduced **Fold Expressions** to process packs directly using operators.

### Fold Operator Matrix
Let `args` be the pack, `op` be a binary operator, and `init` be an initial value.

| Category | Syntax | Expansion Pattern |
|---|---|---|
| **Unary Right Fold** | `(args op ...)` | `args1 op (args2 op (args3 op args4))` |
| **Unary Left Fold** | `(... op args)` | `((args1 op args2) op args3) op args4` |
| **Binary Right Fold**| `(args op ... op init)`| `args1 op (args2 op (args3 op (args4 op init)))` |
| **Binary Left Fold** | `(init op ... op args)`| `(((init op args1) op args2) op args3) op args4` |

---

## 4. Practical Fold Applications

### A. All / Any Conditions (Logical AND / OR)
Verify that all arguments satisfy a specific condition at compile-time:

```cpp
#include <concepts>

template <typename... Args>
bool allPositive(Args... args) {
    // Unary Left Fold using logical AND (&&)
    // Expands to: ((args1 > 0) && (args2 > 0)) && ...
    return (... && (args > 0)); 
}
```

### B. Perfect Forwarding Packs
To forward a pack of arguments to an inner constructor while maintaining value categories (lvalue/rvalue) and constness, use `std::forward` combined with pack expansion:

```cpp
#include <memory>
#include <utility>

template <typename T, typename... Args>
std::unique_ptr<T> createInstance(Args&&... args) {
    // Expand the forwarded arguments pack: std::forward<Args>(args)...
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}
```

---

*<- [[04_Tmpl_Specialization|Template Specialization]] · [[04_Tmpl_SFINAE_and_Type_Traits|SFINAE & Type Traits ->]]*
