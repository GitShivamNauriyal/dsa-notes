---
tags: [cpp, oop, solutions]
links: ["[[03_OOP_Index]]", "[[03_OOP_Problems]]", "[[03_OOP_Tricky]]"]
---

# OOP in C++ -- Solutions

*<- [[03_OOP_Problems|Problems]] · [[03_OOP_Tricky|Tricky OOP Gotchas ->]]*

---

## Tier 1 -- Concept Checks

### Solution 1.1: Virtual Destructors
- **Output**: `~Base()`
- **Explanation**: 
  - Because `Base` does not have a `virtual` destructor, when `delete b` is called on a `Base*` pointer pointing to a `Derived` object, static binding occurs.
  - The compiler only invokes `~Base()`. The derived destructor `~Derived()` is completely bypassed.
  - **Memory Leak**: The heap memory pointed to by `ptr` (allocated in `Derived` constructor) is never freed, causing a memory leak.
  - **Fix**: Declare `virtual ~Base()` in the base class.

---

### Solution 1.2: Calling Virtuals in Constructors
- **Output**: `Parent Init`
- **Explanation**:
  - When object `c` of class `Child` is created, the base class `Parent` constructor is executed first.
  - During the execution of the `Parent` constructor, the `Child` part of the object has not yet been initialized (its members are unconstructed garbage).
  - Therefore, C++ disables virtual dispatch during constructors/destructors. The dynamic type of the object is treated as `Parent` during `Parent`'s constructor. Calling `log()` resolves statically to `Parent::log()`.

---

## Tier 2 -- Implementation

### Solution 2.1: Resolving Diamond Ambiguity

```cpp
#include <string>
#include <iostream>

class Shape {
public:
    std::string color;
    Shape(const std::string& col) : color(col) {}
};

// Inherit virtually to share Shape instance
class Circle : virtual public Shape {
public:
    Circle(const std::string& col) : Shape(col) {}
};

class Square : virtual public Shape {
public:
    Square(const std::string& col) : Shape(col) {}
};

class CircleSquare : public Circle, public Square {
public:
    // D must call grandparent Shape constructor directly!
    CircleSquare(const std::string& col) : Shape(col), Circle(col), Square(col) {}
};

int main() {
    CircleSquare cs("Red");
    std::cout << cs.color << "\n"; // OK: accesses unique shared member directly!
    return 0;
}
```

---

## Tier 3 -- Interview-Level

### Solution 3.1: The Clone Idiom (Virtual Constructors)

1. **How it works**:
   - Define a pure virtual function `clone()` returning a pointer to the base type.
   - Every derived class overrides `clone()`, calling its own copy constructor (e.g. `return new Derived(*this);`).
   - Clients can now duplicate polymorphic objects cleanly: `Base* copy = original->clone();`.

2. **Code Implementation**:

```cpp
#include <string>

class Animal {
public:
    virtual ~Animal() = default;
    virtual Animal* clone() const = 0; // Virtual copy constructor idiom
};

class Dog : public Animal {
    std::string* name;
public:
    Dog(const std::string& n) {
        name = new std::string(n);
    }
    
    ~Dog() override {
        delete name;
    }

    // Copy constructor (for deep copy)
    Dog(const Dog& other) {
        name = new std::string(*other.name);
    }

    // Override clone returning base pointer
    Animal* clone() const override {
        return new Dog(*this); // Invokes Dog copy constructor
    }
};
```

---

### Solution 3.2: Object Slicing
- **Concept**: Object slicing occurs when a derived class object is assigned by value to a base class object. 
- The derived class variables are "sliced away", leaving only the base class portion.

```cpp
class Base {
public:
    virtual void show() { std::cout << "Base\n"; }
};

class Derived : public Base {
public:
    void show() override { std::cout << "Derived\n"; }
};

void demoSlicing() {
    Derived d;
    Base b = d; // Slicing happens here! copy constructor of Base is called.
    b.show();   // Outputs "Base" (polymorphic behavior is lost!)
}
```
- **Why dangerous**: Losing derived state can lead to logic corruption or accessing sliced objects incorrectly in heterogeneous collections.
- **Prevention**: Pass by reference (`Base&`) or pointer (`Base*`), which preserves dynamic binding and vptr mapping.

---

## Tier 4 -- Systems / Placement-Hard

### Solution 4.1: Custom Managed Matrix Class

```cpp
#include <algorithm>
#include <utility>

class Matrix {
    float* data;
    int rows;
    int cols;
public:
    Matrix(int r, int c) : rows(r), cols(c) {
        data = new float[rows * cols]();
    }

    // 1. Destructor
    ~Matrix() {
        delete[] data;
    }

    // 2. Copy Constructor
    Matrix(const Matrix& other) : rows(other.rows), cols(other.cols) {
        data = new float[rows * cols];
        std::copy(other.data, other.data + (rows * cols), data);
    }

    // Helper for Copy and Swap Idiom
    friend void swap(Matrix& first, Matrix& second) noexcept {
        using std::swap;
        swap(first.data, second.data);
        swap(first.rows, second.rows);
        swap(first.cols, second.cols);
    }

    // 3. Copy Assignment Operator (Copy and Swap)
    Matrix& operator=(Matrix other) {
        swap(*this, other);
        return *this;
    }

    // 4. Move Constructor
    Matrix(Matrix&& other) noexcept : data(nullptr), rows(0), cols(0) {
        swap(*this, other);
    }

    // 5. Move Assignment Operator
    Matrix& operator=(Matrix&& other) noexcept {
        Matrix temp(std::move(other));
        swap(*this, temp);
        return *this;
    }

    // Access operator
    float& operator()(int r, int c) {
        return data[r * cols + c];
    }

    const float& operator()(int r, int c) const {
        return data[r * cols + c];
    }
};
```

---

*<- [[03_OOP_Problems|Problems]] · [[03_OOP_Tricky|Tricky OOP Gotchas ->]]*
