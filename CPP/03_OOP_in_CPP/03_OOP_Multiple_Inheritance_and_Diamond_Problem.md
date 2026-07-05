---
tags: [cpp, oop, multiple-inheritance, diamond-problem, virtual-inheritance, memory-layout]
links: ["[[03_OOP_Index]]", "[[03_OOP_Inheritance_and_Virtual_Dispatch]]", "[[03_OOP_Static_vs_Dynamic_Polymorphism_CRTP]]"]
---

# OOP in C++ -- Multiple Inheritance & Diamond Problem

*<- [[03_OOP_Inheritance_and_Virtual_Dispatch|Inheritance & Virtual Dispatch]] · [[03_OOP_Static_vs_Dynamic_Polymorphism_CRTP|Static vs Dynamic & CRTP ->]]*

---

## 1. Multiple Inheritance Memory Layout

When a class inherits from multiple parents, the compiler lays out the parent classes in memory sequentially in the order of their inheritance declaration.

```cpp
class Parent1 { int p1; };
class Parent2 { int p2; };
class Child : public Parent1, public Parent2 { int c; };
```

### Stack Memory Layout of `Child`:
```
   Low Address ──► ┌──────────────────┐
                   │ Parent1::p1      │ (4 bytes)
                   ├──────────────────┤
                   │ Parent2::p2      │ (4 bytes)
                   ├──────────────────┤
                   │ Child::c         │ (4 bytes)
   High Address ──►└──────────────────┘
```

---

## 2. The Diamond Problem

The Diamond Problem is a classic inheritance conflict that occurs when class `D` inherits from both `B` and `C`, and both `B` and `C` share a common ancestor `A`.

```
               A (Grandparent)
              / \
             B   C (Parents)
              \ /
               D (Child)
```

### The Conflict: Dual Memory Paths
Without virtual inheritance, the compiler instantiates two independent copies of the grandparent class `A` inside a single object of class `D`.

#### Memory Layout of `D` (Without Virtual Inheritance):
```
  ┌─────────────────────────┐
  │ A (via B) - int value   │  ◄── Duplicate 1
  ├─────────────────────────┤
  │ B members               │
  ├─────────────────────────┤
  │ A (via C) - int value   │  ◄── Duplicate 2
  ├─────────────────────────┤
  │ C members               │
  ├─────────────────────────┤
  │ D members               │
  └─────────────────────────┘
```
- **Ambiguity**: Accessing `d.value` causes a compiler error because the compiler cannot determine if you want to access the `value` in the `B` path or the `C` path. You have to write `d.B::value` or `d.C::value`.

---

## 3. The Resolution: Virtual Inheritance

Declaring inheritance as **`virtual`** instructs the compiler that the base class must be shared as a single common instance in the final derived class.

```cpp
#include <iostream>

class A {
public:
    int value;
    A(int v) : value(v) {}
};

class B : virtual public A {
public:
    B(int v) : A(v) {}
};

class C : virtual public A {
public:
    C(int v) : A(v) {}
};

class D : public B, public C {
public:
    // Important: D is responsible for initializing A directly!
    D(int v) : A(v), B(v), C(v) {}
};
```

---

## 4. Virtual Inheritance Memory Layout: `vbptr`

To resolve the offsets to a shared base object that is placed dynamically in memory, the compiler inserts a **`vbptr` (Virtual Base Pointer)** into classes `B` and `C`.
- `vbptr` points to an offset table (virtual base table, or `vbtable`) that stores the byte displacement of the shared virtual base `A` relative to `B` or `C`.

#### Memory Layout of `D` (With Virtual Inheritance):
```
  ┌─────────────────────────┐
  │ B::vbptr (offset to A)  ├───────┐
  ├─────────────────────────┤       │
  │ B members               │       │
  ├─────────────────────────┤       │
  │ C::vbptr (offset to A)  ├───────┼──┐
  ├─────────────────────────┤       │  │
  │ C members               │       │  │
  ├─────────────────────────┤       │  │
  │ D members               │       │  │
  ├─────────────────────────┤       │  │
  │ Shared A (int value)    │◄──────┴──┘  (Placed at the bottom of the object)
  └─────────────────────────┘
```

---

## 5. Hierarchical Initialization Responsibility

A crucial language rule under virtual inheritance:
- The **most derived class constructor** (here, class `D`) is **solely responsible** for invoking the constructor of the shared virtual base class `A`.
- The initialization parameters passed to `A` in the constructors of intermediate classes `B` and `C` are **completely ignored** by the compiler.
- If the shared base class `A` lacks a default constructor, the most derived class `D` **must** explicitly call `A`'s constructor in its member initialization list, or the code will fail to compile.

---

*<- [[03_OOP_Inheritance_and_Virtual_Dispatch|Inheritance & Virtual Dispatch]] · [[03_OOP_Static_vs_Dynamic_Polymorphism_CRTP|Static vs Dynamic & CRTP ->]]*
