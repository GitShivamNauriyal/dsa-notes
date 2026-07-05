---
tags: [cpp, oop, inheritance, virtual-dispatch, vtable, vptr, virtual-destructor]
links: ["[[03_OOP_Index]]", "[[03_OOP_Rule_of_Zero_Three_Five]]", "[[03_OOP_Multiple_Inheritance_and_Diamond_Problem]]"]
---

# OOP in C++ -- Inheritance & Virtual Dispatch

*<- [[03_OOP_Rule_of_Zero_Three_Five|Rule of Zero/Three/Five]] · [[03_OOP_Multiple_Inheritance_and_Diamond_Problem|Multiple Inheritance & Diamond ->]]*

---

## 1. Virtual Dispatch Mechanics

In C++, by default, function calls are bound at compile-time (**static binding**). To enable runtime polymorphism (**dynamic binding**), we declare base class functions using the **`virtual`** keyword.
- If a function is virtual, calling it through a base class pointer or reference invokes the version defined in the actual derived object.

---

## 2. Memory Layout: `vptr` and `vtable`

Dynamic binding is implemented under the hood using a table of function pointers known as the **`vtable`** (virtual table) and a pointer to it inside each object known as the **`vptr`** (virtual pointer).

### ASCII Structural Layout

```
    Object on Heap/Stack                    Class VTable
  ┌─────────────────────────┐             ┌─────────────────────────┐
  │ vptr (hidden pointer)  ─┼────────────►│ &Derived::speak()       │
  ├─────────────────────────┤             ├─────────────────────────┤
  │ int data = 42;          │             │ &Base::run()            │
  └─────────────────────────┘             └─────────────────────────┘
```

- **`vtable` (created per-class)**: An array of function pointers containing the addresses of virtual functions for that class.
- **`vptr` (created per-object instance)**: A hidden pointer added by the compiler to the beginning of the object structure. It points to the class's `vtable`.
- **Memory Overhead**:
  - Every object of a class with virtual functions gets a `vptr` (usually 8 bytes on a 64-bit architecture).
  - Every call to a virtual function requires an indirect pointer jump: dereference `vptr` $\to$ offset into `vtable` $\to$ dereference function pointer. This prevents the compiler from performing inline optimizations on these calls.

```cpp
#include <iostream>

class Base {
    int x;
public:
    virtual void func() {}
};

class NonVirtual {
    int x;
};

void demoSizes() {
    // NonVirtual: only size of int (4 bytes, padded to 4)
    std::cout << sizeof(NonVirtual) << "\n"; // Outputs 4
    
    // Base: size of int (4) + size of vptr (8) = 12, padded to 16
    std::cout << sizeof(Base) << "\n";       // Outputs 16
}
```

---

## 3. The Virtual Destructor Rule

> [!WARNING]
> If you delete a derived class object through a pointer to a base class, and the base class lacks a **virtual destructor**, the behavior is **undefined**. In practice, the compiler will only call the base class destructor, leaking any resources allocated by the derived class!

```cpp
#include <iostream>

class Base {
public:
    Base() {}
    // Mandatory virtual destructor for polymorphic base classes
    virtual ~Base() {
        std::cout << "Base Destructor\n";
    }
};

class Derived : public Base {
    int* data;
public:
    Derived() { data = new int[100]; }
    ~Derived() override {
        delete[] data; // Guaranteed to be called if Base has a virtual destructor!
        std::cout << "Derived Destructor\n";
    }
};

void cleanupDemo() {
    Base* ptr = new Derived();
    delete ptr; // Calls Derived destructor first, then Base destructor
}
```

---

*<- [[03_OOP_Rule_of_Zero_Three_Five|Rule of Zero/Three/Five]] · [[03_OOP_Multiple_Inheritance_and_Diamond_Problem|Multiple Inheritance & Diamond ->]]*
