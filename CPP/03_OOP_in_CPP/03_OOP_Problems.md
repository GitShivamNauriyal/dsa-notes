---
tags: [cpp, oop, problems, exercises]
links: ["[[03_OOP_Index]]", "[[03_OOP_Operator_Overloading]]", "[[03_OOP_Solutions]]"]
---

# OOP in C++ -- Problems & Exercises

*<- [[03_OOP_Operator_Overloading|Operator Overloading]] · [[03_OOP_Solutions|Solutions ->]]*

---

## Tier 1 -- Concept Checks

### Question 1.1: Virtual Destructors
What is the output of the following program? If there is undefined behavior or a resource leak, explain why.
```cpp
#include <iostream>

class Base {
public:
    ~Base() { std::cout << "~Base()\n"; }
};

class Derived : public Base {
    int* ptr;
public:
    Derived() { ptr = new int(10); }
    ~Derived() { 
        delete ptr; 
        std::cout << "~Derived()\n"; 
    }
};

int main() {
    Base* b = new Derived();
    delete b;
    return 0;
}
```

---

### Question 1.2: Calling Virtuals in Constructors
Predict the output of this code and explain the underlying C++ mechanics:
```cpp
#include <iostream>

class Parent {
public:
    Parent() { log(); }
    virtual void log() { std::cout << "Parent Init\n"; }
};

class Child : public Parent {
public:
    Child() {}
    void log() override { std::cout << "Child Init\n"; }
};

int main() {
    Child c;
    return 0;
}
```

---

## Tier 2 -- Implementation

### Question 2.1: Resolving Diamond Ambiguity
Write classes `Shape`, `Circle`, `Square`, and `CircleSquare` where:
- `Shape` contains a member variable `std::string color;` and a constructor initializing it.
- `Circle` and `Square` inherit from `Shape`.
- `CircleSquare` inherits from both `Circle` and `Square`.
Implement constructors and verify that a `CircleSquare` object can access `color` directly without scope resolution conflicts.

---

## Tier 3 -- Interview-Level

### Question 3.1: The Clone Idiom (Virtual Constructors)
C++ constructors cannot be declared `virtual`. However, we frequently need to create a copy of an object when we only have a pointer to its base interface class (without knowing its concrete type).
1. Explain how to implement the **Virtual Constructor / Clone Idiom** to solve this.
2. Write a base class `Animal` with a virtual function `Animal* clone() const = 0`, and implement it in a derived class `Dog` that manages a heap-allocated name string.

---

### Question 3.2: Object Slicing
Explain the concept of **Object Slicing** with a code example. How does it happen, why is it dangerous in systems code, and how do references/pointers prevent it?

---

## Tier 4 -- Systems / Placement-Hard

### Question 4.1: Custom Managed Matrix Class
Write a custom `Matrix` class that represents a 2D grid of floats. The class must manage its own heap-allocated 1D flat array memory.
1. Implement constructors, destructor, and operator access `operator()(int r, int c)`.
2. Adhere to the **Rule of Five**: implement copy constructor, copy assignment (using the copy-and-swap idiom to ensure strong exception safety), move constructor, and move assignment.
3. Validate that no double-frees or memory leaks occur.

---

*<- [[03_OOP_Operator_Overloading|Operator Overloading]] · [[03_OOP_Solutions|Solutions ->]]*
