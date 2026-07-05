---
tags: [cpp, oop, tricky, object-slicing, default-arguments, nvi]
links: ["[[03_OOP_Index]]", "[[03_OOP_Solutions]]", "[[../04_Templates_and_Generics/04_Tmpl_Index]]"]
---

# OOP in C++ -- Tricky & Higher-Order

*<- [[03_OOP_Solutions|Solutions]] · [[../04_Templates_and_Generics/04_Tmpl_Index|Templates & Generics ->]]*

---

## 1. Default Arguments with Virtual Functions Gotcha

**The Rule**: In C++, virtual function dispatch is resolved **dynamically at runtime**, but default parameter values are bound **statically at compile-time** based on the static type of the pointer/reference.

```cpp
#include <iostream>

class Base {
public:
    virtual void show(int x = 100) {
        std::cout << "Base: " << x << "\n";
    }
};

class Derived : public Base {
public:
    void show(int x = 200) override {
        std::cout << "Derived: " << x << "\n";
    }
};

void demoDefault() {
    Base* ptr = new Derived();
    // Static Type of ptr: Base*
    // Dynamic Type of ptr: Derived*
    
    // Call resolves to Derived::show(), but gets the default argument of Base (100)!
    ptr->show(); // Outputs "Derived: 100"!
}
```

### Best Practice:
Never override default parameter values in virtual functions. If you must use defaults, use the **Non-Virtual Interface (NVI)** pattern.

---

## 2. Non-Virtual Interface (NVI) Pattern

**Concept**: Expose public non-virtual wrapper methods that internalize pre/post execution checks, and call private or protected virtual methods to do the actual work.
- This separates interface (public non-virtual) from implementation customizability (private virtual).

```cpp
class Parser {
public:
    // Public non-virtual interface
    void parse() {
        // Enforce pre-parse steps (e.g. logging, state locks)
        doParse(); // calls virtual implementation
        // Enforce post-parse steps
    }
    
    virtual ~Parser() = default;

private:
    // Private virtual implementation
    virtual void doParse() = 0;
};

class XmlParser : public Parser {
private:
    void doParse() override {
        // Concrete XML parsing details
    }
};
```

---

## 3. Object Slicing Under the Hood
When a `Derived` object is copied into a `Base` object by value:
- The base class copy constructor is called.
- The compiler copies the base member variables.
- The `vptr` is set to point to the `Base` class's `vtable`, completely stripping the derived class identity and custom virtual methods.

---

## Recap of Tricky OOP Gotchas

| Gotcha | Theme |
|---|---|
| **Default Arguments + Virtuals** | Methods resolve dynamically, but default arguments bind statically at compile-time based on pointer type. |
| **NVI Pattern** | Public interface remains static and non-virtual; implementation customizability kept private and virtual. |
| **Object Slicing** | Value-based assignment strips derived variables and resets the `vptr` to the base class vtable. |

---

*<- [[03_OOP_Solutions|Solutions]] · [[../04_Templates_and_Generics/04_Tmpl_Index|Templates & Generics ->]]*
