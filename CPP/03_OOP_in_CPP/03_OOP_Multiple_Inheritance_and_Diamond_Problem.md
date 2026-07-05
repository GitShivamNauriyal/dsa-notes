---
tags: [cpp, oop, multiple-inheritance, diamond-problem, virtual-inheritance]
links: ["[[03_OOP_Index]]", "[[03_OOP_Inheritance_and_Virtual_Dispatch]]", "[[03_OOP_Static_vs_Dynamic_Polymorphism_CRTP]]"]
---

# OOP in C++ -- Multiple Inheritance & Diamond Problem

*<- [[03_OOP_Inheritance_and_Virtual_Dispatch|Inheritance & Virtual Dispatch]] · [[03_OOP_Static_vs_Dynamic_Polymorphism_CRTP|Static vs Dynamic & CRTP ->]]*

---

## 1. Multiple Inheritance Layout

C++ allows a class to inherit from multiple parent classes. However, this introduces complexity in memory layouts and names resolution.

---

## 2. The Diamond Problem

The **Diamond Problem** occurs when two classes `B` and `C` inherit from class `A`, and a class `D` inherits from both `B` and `C`.

### ASCII Inheritance Tree

```
               A (Grandparent)
              / \
             B   C (Parents)
              \ /
               D (Child)
```

Without special treatment, an object of class `D` contains **two separate instances** of class `A`:
- One instance of `A` inherited through `B`.
- Another instance of `A` inherited through `C`.

This duplicate allocation leads to:
1. **Memory waste**: Doubling grandparent class variables.
2. **Ambiguity**: Accessing `A`'s members through `D` fails compile checks because the compiler doesn't know whether to route via path `B` or path `C`.

---

## 3. The Resolution: Virtual Inheritance

To resolve the duplicate instance issue, we instruct the compiler to share the base class instance using the **`virtual`** keyword in the inheritance declaration.

```cpp
#include <iostream>

class A {
public:
    int value = 99;
    void print() { std::cout << value << "\n"; }
};

// Declaring virtual inheritance ensures A is a shared virtual base
class B : virtual public A {};
class C : virtual public A {};

class D : public B, public C {};

void demoDiamond() {
    D obj;
    // Without 'virtual' keyword above:
    // obj.print(); // COMPILER ERROR: Request for member 'print' is ambiguous!
    
    // With virtual inheritance:
    obj.print(); // OK: Prints 99 (calls the single shared instance of A)
}
```

### Under the Hood: Virtual Base Pointers (`vbptr`)
When virtual inheritance is used, the compiler inserts a virtual base pointer (`vbptr`) into classes `B` and `C`.
- `vbptr` points to an offset table that tells the class where to find the shared virtual base `A` in memory.
- During constructor calls of `D`, the shared grandparent `A` is initialized directly by `D`'s constructor, completely bypassing `B` and `C`'s constructors for `A`.

---

*<- [[03_OOP_Inheritance_and_Virtual_Dispatch|Inheritance & Virtual Dispatch]] · [[03_OOP_Static_vs_Dynamic_Polymorphism_CRTP|Static vs Dynamic & CRTP ->]]*
