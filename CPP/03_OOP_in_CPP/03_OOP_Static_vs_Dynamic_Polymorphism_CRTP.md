---
tags: [cpp, oop, polymorphism, static-polymorphism, crtp, mixins]
links: ["[[03_OOP_Index]]", "[[03_OOP_Multiple_Inheritance_and_Diamond_Problem]]", "[[03_OOP_Operator_Overloading]]", "[[../04_Templates_and_Generics/04_Tmpl_Index]]"]
---

# OOP in C++ -- Static vs Dynamic Polymorphism & CRTP

*<- [[03_OOP_Multiple_Inheritance_and_Diamond_Problem|Multiple Inheritance & Diamond]] · [[03_OOP_Operator_Overloading|Operator Overloading ->]]*

---

## 1. Deep Comparison: Static vs Dynamic Polymorphism

Polymorphism allows processing different types through a unified interface. C++ supports this via compile-time (static) or runtime (dynamic) binding.

| Dimension | Dynamic Polymorphism | Static Polymorphism |
|---|---|---|
| **Mechanism** | Class Inheritance & `virtual` functions | Templates, Overloading, and CRTP |
| **Binding Time** | Runtime (Virtual table lookup) | Compile-time (Template expansion) |
| **Memory Cost** | adds `vptr` (8 bytes per object) + `vtable` | 0 bytes (Empty Base Class Optimization) |
| **Speed Cost** | Indirect call indirection (prevents inlining)| Zero-overhead (direct function inline calls) |
| **Binary Size** | Smaller (shared code in base class methods) | Can lead to code bloat (template instantiations) |
| **Flexibility** | Heterogeneous collections (`std::vector<Base*>`) | Homogeneous collections |

### De-virtualization Optimization
In modern compilers, if the compiler can prove the exact runtime type of an object (e.g. `Base* ptr` points to a local variable `Derived d;` on the stack whose type doesn't change, or the subclass is marked `final`), the compiler bypasses the virtual table lookup entirely. This optimization is called **de-virtualization** and restores static call speed to dynamic methods.

---

## 2. Curiously Recurring Template Pattern (CRTP)

**CRTP** implements static polymorphism by deriving a class from a template base class, passing the derived class itself as a template parameter.

### Static Dispatch Cast
Because the base class knows the exact type of the derived class at compile-time, we can static-cast `this` (which is of type `Base<Derived>*`) directly to `Derived*` to invoke the derived methods.

```cpp
#include <iostream>

template <typename Derived>
class Logger {
public:
    void log(const std::string& msg) {
        // Static Cast resolving to Derived class method at compile-time
        static_cast<Derived*>(this)->write(msg);
    }
};

class ConsoleLogger : public Logger<ConsoleLogger> {
public:
    void write(const std::string& msg) {
        std::cout << "[Console]: " << msg << "\n";
    }
};
```

---

## 3. CRTP Mixins (Injecting Common Behavior)

A powerful use case of CRTP is **mixins**—injecting common, derived operations into classes automatically if they define a core set of primary functions.

### Example: Auto-generating Comparison Operators
Suppose we want to auto-generate operators `>`, `<=`, and `>=` automatically if a class implements `<`.

```cpp
template <typename Derived>
struct Comparable {
    // Inject > operator based on derived class '<' operator
    friend bool operator>(const Derived& lhs, const Derived& rhs) {
        return rhs < lhs; // delegates to derived class operator<
    }

    friend bool operator<=(const Derived& lhs, const Derived& rhs) {
        return !(lhs > rhs);
    }

    friend bool operator>=(const Derived& lhs, const Derived& rhs) {
        return !(lhs < rhs);
    }
};

// Derived class inherits Comparable and passes itself
class Quantity : public Comparable<Quantity> {
    int value;
public:
    explicit Quantity(int val) : value(val) {}

    // Implement the core operator<
    bool operator<(const Quantity& other) const {
        return this->value < other.value;
    }
};

void demoComparable() {
    Quantity q1(10);
    Quantity q2(20);

    // Operator > and <= work automatically! No runtime virtual overhead.
    if (q1 > q2) {
        // ...
    }
}
```

---

## 4. Size Overhead Verification

Under the C++ **Empty Base Class Optimization (EBCO)**, if a base class contains no non-static member variables and no virtual functions, the compiler optimizes away its size.
- A CRTP base template is an empty class.
- Therefore, inheriting from a CRTP base class adds **exactly 0 bytes** to the size of the derived class.

```cpp
#include <iostream>

template <typename Derived>
struct EmptyCRTPBase {};

struct Counter : public EmptyCRTPBase<Counter> {
    int count; // 4 bytes
};

void printCRTPSize() {
    // Outputs 4 bytes! The CRTP base class adds no memory overhead.
    std::cout << "Size: " << sizeof(Counter) << "\n"; 
}
```

---

*<- [[03_OOP_Multiple_Inheritance_and_Diamond_Problem|Multiple Inheritance & Diamond]] · [[03_OOP_Operator_Overloading|Operator Overloading ->]]*
