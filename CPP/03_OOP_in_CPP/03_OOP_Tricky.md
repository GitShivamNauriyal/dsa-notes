---
tags: [cpp, oop, tricky, object-slicing, default-arguments, nvi, constructor-virtuals]
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

## 3. Object Slicing Under the Hood & STL Containers

When a `Derived` object is copied into a `Base` object by value, its derived components are stripped away, and its `vptr` is re-pointed to the `Base` class's `vtable`.

### Container Slicing Trap:
If you store polymorphism by value in standard library containers, slicing occurs silently.

```cpp
#include <vector>
#include <memory>

class Base {
public:
    virtual void sayHi() const { std::cout << "Base\n"; }
};
class Derived : public Base {
public:
    void sayHi() const override { std::cout << "Derived\n"; }
};

void demoVectorSlice() {
    std::vector<Base> vec;
    vec.push_back(Derived()); // Slicing! Copy constructor of Base is called
    
    vec[0].sayHi(); // Outputs: "Base" (polymorphic state is completely lost!)
}
```

### The Solutions:
1. **Pointers**: Use `std::vector<std::unique_ptr<Base>>` (exclusive heap ownership).
2. **References**: Use `std::vector<std::reference_wrapper<Base>>` from `<functional>` to store reference semantics without heap allocations.

---

## 4. Virtual Call Resolution in Constructors & Destructors

During the construction of a base class, the object is not yet a derived class instance.
- The `vptr` inside the base class constructor points to the **Base class VTable**.
- Calling a virtual function resolves to the base class implementation.
- If the virtual function is **pure virtual** (`= 0`) and has no definition, this triggers undefined behavior or immediate linker crashes (`pure virtual method called` runtime error).

```cpp
class Component {
public:
    Component() {
        init(); // DANGEROUS: resolves to Component::init(), not Derived!
    }
    virtual void init() { std::cout << "Base Init\n"; }
};

class GraphicsComponent : public Component {
public:
    void init() override { std::cout << "Derived Graphics Init\n"; }
};
```

---

## Recap of Tricky OOP Gotchas

| Gotcha | Theme |
|---|---|
| **Default Arguments + Virtuals** | Methods resolve dynamically, but default arguments bind statically at compile-time based on pointer type. |
| **NVI Pattern** | Public interface remains static and non-virtual; implementation customizability kept private and virtual. |
| **Object Slicing** | Value-based assignment strips derived variables and resets the `vptr` to the base class vtable. |
| **Constructor Virtuals** | Virtual dispatch is disabled inside constructors and destructors because the derived state is not yet constructed or is already destroyed. |

---

*<- [[03_OOP_Solutions|Solutions]] · [[../04_Templates_and_Generics/04_Tmpl_Index|Templates & Generics ->]]*
