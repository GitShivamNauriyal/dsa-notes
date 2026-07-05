---
tags: [cpp, oop, polymorphism, static-polymorphism, crtp]
links: ["[[03_OOP_Index]]", "[[03_OOP_Multiple_Inheritance_and_Diamond_Problem]]", "[[03_OOP_Operator_Overloading]]", "[[../04_Templates_and_Generics/04_Tmpl_Index]]"]
---

# OOP in C++ -- Static vs Dynamic Polymorphism & CRTP

*<- [[03_OOP_Multiple_Inheritance_and_Diamond_Problem|Multiple Inheritance & Diamond]] · [[03_OOP_Operator_Overloading|Operator Overloading ->]]*

---

## 1. Static vs Dynamic Polymorphism

| Dimension | Dynamic Polymorphism | Static Polymorphism |
|---|---|---|
| **Mechanism** | Inheritance & `virtual` functions | Templates & Overloading |
| **Binding Time** | Runtime | Compile-time |
| **Overhead** | Memory (`vptr`) + Indirection jump | None (Zero-cost abstraction) |
| **Compiler Optimization**| Limited (cannot inline virtual calls) | Full (inlining, constant folding) |
| **Flexibility** | Heterogeneous containers (e.g. `vector<Base*>`) | Homogeneous collections |

---

## 2. Curiously Recurring Template Pattern (CRTP)

**CRTP** is a C++ design pattern that implements static polymorphism.

### Structure:
A class `Derived` inherits from a template base class `Base`, passing `Derived` itself as the template argument:

```cpp
template <typename Derived>
class Base {
    // Base implementation details...
};

class Derived : public Base<Derived> {
    // Derived implementation details...
};
```

### Static Dispatch Mechanism:
Inside `Base`, we use `static_cast` to cast the `Base*` pointer to `Derived*`. This allows the base class to call derived class methods without using virtual functions!

```cpp
#include <iostream>

template <typename Derived>
class Writer {
public:
    void write(const std::string& msg) {
        // Static dispatch: cast this to Derived and call implementation
        static_cast<Derived*>(this)->writeImpl(msg);
    }
};

class ConsoleWriter : public Writer<ConsoleWriter> {
public:
    void writeImpl(const std::string& msg) {
        std::cout << "[Console]: " << msg << "\n";
    }
};

class FileWriter : public Writer<FileWriter> {
public:
    void writeImpl(const std::string& msg) {
        // e.g. write to file stream
        std::cout << "[File]: " << msg << "\n";
    }
};

template <typename T>
void executeWrite(Writer<T>& writer, const std::string& msg) {
    writer.write(msg); // Resolved at compile-time!
}

void demoCRTP() {
    ConsoleWriter cw;
    FileWriter fw;
    
    executeWrite(cw, "Hello CRTP"); // Calls ConsoleWriter::writeImpl
    executeWrite(fw, "Log data");   // Calls FileWriter::writeImpl
}
```

### Why use CRTP?
1. **Performance**: No `vptr` memory overhead, and calls can be inlined by the compiler.
2. **Interface compliance**: Enforces derived classes to implement specific methods, throwing compile-time errors if they don't.
3. **Mixins / Code reuse**: Add functionality to derived classes automatically by defining common wrapper methods in the base template.

---

*<- [[03_OOP_Multiple_Inheritance_and_Diamond_Problem|Multiple Inheritance & Diamond]] · [[03_OOP_Operator_Overloading|Operator Overloading ->]]*
