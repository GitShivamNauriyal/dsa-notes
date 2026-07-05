---
tags: [cpp, templates, template-specialization, partial-specialization, full-specialization, member-specialization]
links: ["[[04_Tmpl_Index]]", "[[04_Tmpl_Function_and_Class_Templates]]", "[[04_Tmpl_Variadic_Templates]]"]
---

# Templates & Generics -- Template Specialization

*<- [[04_Tmpl_Function_and_Class_Templates|Function & Class Templates]] · [[04_Tmpl_Variadic_Templates|Variadic Templates ->]]*

---

## 1. Full Specialization

**Full Specialization** (also called explicit specialization) defines a completely custom implementation of a class or function template for a single, specific set of template arguments.

- **Syntax**: Write `template <>` and place the target types in brackets after the template name.

```cpp
#include <iostream>
#include <vector>

// Primary Class Template
template <typename T>
class DataPrinter {
public:
    void print(const T& val) {
        std::cout << "Generic Print: " << val << "\n";
    }
};

// Full Specialization for std::vector<int>
template <>
class DataPrinter<std::vector<int>> {
public:
    void print(const std::vector<int>& vec) {
        std::cout << "Vector Print: [ ";
        for (int x : vec) std::cout << x << " ";
        std::cout << "]\n";
    }
};
```

---

## 2. Partial Specialization (Classes Only)

**Partial Specialization** allows you to specialize a template for a *family of types* (such as all pointer types, all reference types, or all types matching a certain structure) while keeping some parameters generic.

> [!IMPORTANT]
> The C++ standard **prohibits** the partial specialization of function templates. Only class templates (and variable templates since C++14) can be partially specialized.

### Partial Specialization Families

```cpp
// Primary Class Template
template <typename T, typename U>
class TypeInspector {
public:
    void inspect() { std::cout << "Primary Generic\n"; }
};

// 1. Specialize only one parameter:
template <typename T>
class TypeInspector<T, int> {
public:
    void inspect() { std::cout << "Partial: Second parameter is int\n"; }
};

// 2. Specialize for pointer families:
template <typename T, typename U>
class TypeInspector<T*, U*> {
public:
    void inspect() { std::cout << "Partial: Both parameters are raw pointers\n"; }
};

// 3. Specialize for const types:
template <typename T, typename U>
class TypeInspector<const T, U> {
public:
    void inspect() { std::cout << "Partial: First parameter is const\n"; }
};
```

---

## 3. Why Functions Can't Be Partially Specialized

Function templates participate in **overload resolution**, which is already a complex multi-step process in C++. Allowing partial specialization of function templates would create conflicts with overloading (e.g. should a compiler choose an overload or a partial specialization?).

### The Correct Workaround: Overloading

Instead of trying to partially specialize a function template, declare an **overloaded** function template.

```cpp
#include <iostream>

// Primary function template
template <typename T>
void inspectValue(T val) {
    std::cout << "Primary Generic Value: " << val << "\n";
}

// Workaround: Overload for all pointer types
template <typename T>
void inspectValue(T* ptr) {
    if (ptr) {
        std::cout << "Pointer Overload pointing to: " << *ptr << "\n";
    }
}
```

---

## 4. Specializing Individual Member Functions

If you only need to customize a single method of a class template for a specific type, you can specialize **just that member function** instead of replicating the entire class definition.

```cpp
#include <iostream>
#include <string>

template <typename T>
class Processor {
public:
    void setup() {
        std::cout << "Generic Setup\n";
    }
    void execute() {
        std::cout << "Generic Execution\n";
    }
};

// Specialize ONLY the setup() member function for std::string
template <>
void Processor<std::string>::setup() {
    std::cout << "String Custom Setup\n";
}

void demoMemberSpec() {
    Processor<int> p1;
    p1.setup();   // Outputs: Generic Setup
    p1.execute(); // Outputs: Generic Execution

    Processor<std::string> p2;
    p2.setup();   // Outputs: String Custom Setup (Specialized!)
    p2.execute(); // Outputs: Generic Execution (Uses primary)
}
```

---

*<- [[04_Tmpl_Function_and_Class_Templates|Function & Class Templates]] · [[04_Tmpl_Variadic_Templates|Variadic Templates ->]]*
