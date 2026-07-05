---
tags: [cpp, templates, variadic-templates, parameter-packs, fold-expressions]
links: ["[[04_Tmpl_Index]]", "[[04_Tmpl_Specialization]]", "[[04_Tmpl_SFINAE_and_Type_Traits]]"]
---

# Templates & Generics -- Variadic Templates

*<- [[04_Tmpl_Specialization|Template Specialization]] · [[04_Tmpl_SFINAE_and_Type_Traits|SFINAE & Type Traits ->]]*

---

## 1. Parameter Packs (C++11)

**Variadic Templates** accept any number of arguments of any type. 
- **`typename... Args`**: Declares a **template parameter pack**.
- **`Args... args`**: Declares a **function parameter pack**.

```cpp
#include <iostream>

// Base case: terminates recursion when no arguments are left
void printList() {
    std::cout << "\n";
}

// Recursive step: extracts the first argument, recurses on the rest
template <typename T, typename... Args>
void printList(T first, Args... rest) {
    std::cout << first << " ";
    printList(rest...); // Expand the pack
}

void demoVariadic() {
    printList(1, 2.5, "Hello", 'A'); // Prints: 1 2.5 Hello A
}
```

---

## 2. Fold Expressions (C++17)

To print or compute values with variadic templates prior to C++17, we had to write tedious recursive functions with base-case overloads. C++17 introduced **Fold Expressions** to expand parameter packs directly using binary/unary operators.

### Fold Expression Syntax:
Let the parameter pack be `args`, and the operator be `op` (e.g. `+`, `,`, `<<`).

| Type | Syntax | Expanded Form |
|---|---|---|
| **Unary Right Fold** | `(args op ...)` | `args1 op (args2 op (args3 op args4))` |
| **Unary Left Fold** | `(... op args)` | `((args1 op args2) op args3) op args4` |
| **Binary Right Fold**| `(args op ... op init)`| `args1 op (args2 op (args3 op (args4 op init)))` |
| **Binary Left Fold** | `(init op ... op args)`| `(((init op args1) op args2) op args3) op args4` |

---

## 3. Fold Expression Examples

### A. Variadic Sum (Unary Left Fold)
```cpp
template <typename... Args>
auto sumAll(Args... args) {
    return (... + args); // Unary Left Fold: expands to ((args1 + args2) + args3) + ...
}

void demoSum() {
    int sum = sumAll(1, 2, 3, 4, 5); // Returns 15
}
```

### B. Variadic Print (Binary Left Fold with stream `<<`)
```cpp
#include <iostream>

template <typename... Args>
void foldPrint(Args... args) {
    // Binary Left Fold using std::cout as the initial value
    // Expands to: ((std::cout << args1) << args2) << ...
    (std::cout << ... << args) << "\n";
}

void demoFoldPrint() {
    foldPrint("Data:", 42, 3.14, 'Z');
}
```

---

*<- [[04_Tmpl_Specialization|Template Specialization]] · [[04_Tmpl_SFINAE_and_Type_Traits|SFINAE & Type Traits ->]]*
