---
tags: [cpp, oop, inheritance, virtual-dispatch, vtable, vptr, virtual-destructor, covariant-return, override-final]
links: ["[[03_OOP_Index]]", "[[03_OOP_Rule_of_Zero_Three_Five]]", "[[03_OOP_Multiple_Inheritance_and_Diamond_Problem]]"]
---

# OOP in C++ -- Inheritance & Virtual Dispatch

*<- [[03_OOP_Rule_of_Zero_Three_Five|Rule of Zero/Three/Five]] · [[03_OOP_Multiple_Inheritance_and_Diamond_Problem|Multiple Inheritance & Diamond ->]]*

---

## 1. Virtual Dispatch Mechanics

In C++, dynamic binding (invoking a function based on the runtime type of an object rather than its compile-time pointer type) is enabled using the **`virtual`** keyword.
- If a method is marked `virtual` in the base class, it remains virtual for all subsequent subclasses in the hierarchy.
- **Dynamic Binding Trigger**: Dynamic dispatch is triggered **only** when calling the function through a base pointer (`Base*`) or reference (`Base&`). Calling it on a value object (e.g., `Base b = Derived();`) triggers object slicing and runs static binding.

---

## 2. Under the Hood: `vptr` and `vtable` Resolution

The compiler implements dynamic binding using two structures:
1. **`vtable` (Virtual Table)**: A static table created **once per class** that contains an array of function pointers pointing to the virtual methods of that class.
2. **`vptr` (Virtual Pointer)**: A hidden pointer added **to each object instance** at offset 0. It is initialized in the constructor to point to the class's `vtable`.

### Step-by-Step VTable Overwrite Mechanics
Let's see how the compiler populates the `vtable` when subclassing:

1. **Base Class Compilation**:
   - Compiler generates `Base VTable` containing:
     - Index 0: `&Base::f()`
     - Index 1: `&Base::g()`
2. **Derived Class Compilation**:
   - Compiler copies `Base VTable` entries.
   - If `Derived` overrides `f()` but inherits `g()`, the compiler replaces index 0's pointer with `&Derived::f()`, but leaves index 1 pointing to `&Base::g()`.

```
  Base Object (on stack)                     Base VTable (static)
  ┌──────────────────┐                     ┌─────────────────────┐
  │ vptr             ├────────────────────►│ Index 0: &Base::f() │
  ├──────────────────┤                     ├─────────────────────┤
  │ int base_data;   │                     │ Index 1: &Base::g() │
  └──────────────────┘                     └─────────────────────┘

  Derived Object (on stack)                  Derived VTable (static)
  ┌──────────────────┐                     ┌────────────────────────┐
  │ vptr             ├────────────────────►│ Index 0: &Derived::f() │ (Overwritten)
  ├──────────────────┤                     ├────────────────────────┤
  │ int base_data;   │                     │ Index 1: &Base::g()    │ (Inherited)
  ├──────────────────┤                     └────────────────────────┘
  │ int derived_data;│
  └──────────────────┘
```

- **Call Indirection overhead**: A virtual call `ptr->f()` compiles to:
  1. Retrieve `vptr` from `ptr`: `void** vtable = *(void***)ptr;`
  2. Retrieve function pointer at index 0: `void (*func_ptr)() = (void (*)())vtable[0];`
  3. Invoke function pointer: `func_ptr();`
  - This extra pointer dereference blocks compilers from performing **inlining optimizations**, introducing a minor latency penalty.

---

## 3. The `override` and `final` Specifiers (C++11)

### `override`
Ensures that a derived class function signature matches a virtual function declared in the base class. Without `override`, subtle mismatches in constness or parameters compile as new overloaded functions instead of overrides, leading to silent bugs.

```cpp
class Base {
public:
    virtual void process(const std::string& str) const {}
};

class Derived : public Base {
public:
    // Without 'override', this compiles but is a new function (parameter lacks const)!
    // With 'override', compiler throws error, alerting us to the signature mismatch!
    void process(std::string& str) const override {} 
};
```

### `final`
Prevents further overriding of a virtual method, or blocks inheritance of a class entirely (allowing compiler to optimize and de-virtualize calls, restoring static speed).

```cpp
class FinalClass final {}; // Cannot be inherited

// class SubClass : public FinalClass {}; // COMPILER ERROR!
```

---

## 4. Covariant Return Types

An exception to the rule that virtual function overrides must have identical return types:
- A virtual function in a derived class can return a **pointer or reference to a class derived from the type returned by the base class**.

```cpp
class BaseProduct {};
class DerivedProduct : public BaseProduct {};

class Factory {
public:
    virtual BaseProduct* create() { return new BaseProduct(); }
};

class DerivedFactory : public Factory {
public:
    // Covariant Return Type: returning DerivedProduct* instead of BaseProduct*
    DerivedProduct* create() override { return new DerivedProduct(); }
};
```

---

## 5. The Virtual Destructor Mandate

If a derived class object is deleted through a base class pointer `Base*`, and the base class lacks a **virtual destructor**, the compiler executes static binding, calling only `~Base()`. The derived class destructor body is never run, leaking all derived member allocations.

```cpp
#include <iostream>

class Base {
public:
    Base() {}
    // Declaring destructor virtual ensures the entire hierarchy cleans up properly
    virtual ~Base() {
        std::cout << "Base Cleaned\n";
    }
};

class Derived : public Base {
    int* buffer;
public:
    Derived() { buffer = new int[500]; }
    ~Derived() override {
        delete[] buffer; // Safely freed during Base* deletion
        std::cout << "Derived Cleaned\n";
    }
};
```

---

*<- [[03_OOP_Rule_of_Zero_Three_Five|Rule of Zero/Three/Five]] · [[03_OOP_Multiple_Inheritance_and_Diamond_Problem|Multiple Inheritance & Diamond ->]]*
