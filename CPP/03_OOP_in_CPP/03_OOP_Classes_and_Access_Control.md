---
tags: [cpp, oop, class, struct, friend]
links: ["[[03_OOP_Index]]", "[[03_OOP_Constructors_Destructors_and_RAII]]"]
---

# OOP in C++ -- Classes & Access Control

*<- [[03_OOP_Index|Index]] · [[03_OOP_Constructors_Destructors_and_RAII|Constructors & RAII ->]]*

---

## 1. `class` vs `struct` in C++

In C++, `class` and `struct` are virtually identical except for two default settings:

1. **Default Member Access**:
   - `class` members are `private` by default.
   - `struct` members are `public` by default.
2. **Default Inheritance Access**:
   - A `class` inherits privately by default: `class Derived : Base` is equivalent to `class Derived : private Base`.
   - A `struct` inherits publicly by default: `struct Derived : Base` is equivalent to `struct Derived : public Base`.

```cpp
struct Point {
    int x; // public by default
    int y;
};

class Circle {
    int radius; // private by default
public:
    int getRadius() const { return radius; }
};
```

---

## 2. Access Specifiers

Access specifiers control visibility of members outside the class boundary:
- **`public`**: Accessible by any code that can see the class.
- **`private`**: Accessible only by member functions and friends of the class.
- **`protected`**: Accessible by member functions, friends, and derived classes.

### Inheritance Access Control
Inheritance modifiers restrict the maximum visibility of base class members in the derived class:
- **`public` inheritance**: `public` -> `public`, `protected` -> `protected`, `private` -> inaccessible. (Preserves "IS-A" relationship).
- **`protected` inheritance**: `public`/`protected` -> `protected`, `private` -> inaccessible.
- **`private` inheritance**: `public`/`protected` -> `private`, `private` -> inaccessible. (Used for "implemented-in-terms-of").

---

## 3. Friend Classes and Functions

A `friend` declaration allows external functions or classes to bypass access specifiers and directly read/write `private` and `protected` members.

### Properties of Friendships:
1. **Not Symmetric**: If A is a friend of B, it does not mean B is a friend of A.
2. **Not Transitive**: If A is a friend of B, and B is a friend of C, A is not a friend of C.
3. **Not Inherited**: A friend of a base class does not automatically become a friend of a derived class.

```cpp
#include <iostream>

class Engine {
    int temperature = 90;
    
    // Declare external function as friend
    friend void coolEngine(Engine& eng);
    // Declare class as friend
    friend class Mechanic;
};

void coolEngine(Engine& eng) {
    eng.temperature -= 10; // Accesses private member temperature directly
}

class Mechanic {
public:
    void diagnostic(const Engine& eng) {
        std::cout << eng.temperature << "\n"; // Accesses private member
    }
};
```

---

*<- [[03_OOP_Index|Index]] · [[03_OOP_Constructors_Destructors_and_RAII|Constructors & RAII ->]]*
