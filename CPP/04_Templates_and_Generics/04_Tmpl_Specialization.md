---
tags: [cpp, templates, template-specialization, partial-specialization, full-specialization]
links: ["[[04_Tmpl_Index]]", "[[04_Tmpl_Function_and_Class_Templates]]", "[[04_Tmpl_Variadic_Templates]]"]
---

# Templates & Generics -- Template Specialization

*<- [[04_Tmpl_Function_and_Class_Templates|Function & Class Templates]] · [[04_Tmpl_Variadic_Templates|Variadic Templates ->]]*

---

## 1. Full Specialization

**Full Specialization** provides a custom implementation of a template when the template arguments match a specific set of types.

- **Syntax**: Use empty template brackets `template <>` and specify the concrete types in the class/function signature name.

```cpp
#include <iostream>

// Primary Template
template <typename T>
class Storage {
    T val;
public:
    Storage(T v) : val(v) {}
    void print() { std::cout << "Generic storage: " << val << "\n"; }
};

// Full Specialization for bool (to optimize storage space, like std::vector<bool>)
template <>
class Storage<bool> {
    unsigned char val : 1; // bit-field optimization
public:
    Storage(bool v) : val(v) {}
    void print() { std::cout << "Bool specialized storage: " << (val ? "true" : "false") << "\n"; }
};
```

---

## 2. Partial Specialization (Classes Only)

**Partial Specialization** allows you to specialize a template for a **family of types** (e.g. all pointer types, all vectors) while keeping the template parameters generic.

- **Rule**: Only class templates can be partially specialized. Function templates **cannot** be partially specialized (they must use function overloading instead).

```cpp
// Primary Class Template
template <typename T, typename U>
class Pair {
public:
    void print() { std::cout << "Generic Pair\n"; }
};

// Partial Specialization: Second argument is specialized to int
template <typename T>
class Pair<T, int> {
public:
    void print() { std::cout << "Partial: Second is int\n"; }
};

// Partial Specialization: Specialized for pointer types
template <typename T, typename U>
class Pair<T*, U*> {
public:
    void print() { std::cout << "Partial: Both are pointers\n"; }
};
```

---

## 3. Function Templates Overloading Workaround

Because function templates do not support partial specialization, attempting it leads to compiler errors. The correct C++ workaround is **function overloading**.

```cpp
#include <iostream>

// Primary function template
template <typename T>
void debugPrint(T val) {
    std::cout << "Generic: " << val << "\n";
}

// Workaround for partial specialization: overload for all pointers
template <typename T>
void debugPrint(T* ptr) {
    if (ptr) {
        std::cout << "Pointer deref: " << *ptr << "\n";
    }
}
```

---

*<- [[04_Tmpl_Function_and_Class_Templates|Function & Class Templates]] · [[04_Tmpl_Variadic_Templates|Variadic Templates ->]]*
