---
tags: [cpp, oop, operator-overloading, prefix-postfix, spaceship-operator, subscript-operator]
links: ["[[03_OOP_Index]]", "[[03_OOP_Static_vs_Dynamic_Polymorphism_CRTP]]", "[[03_OOP_Problems]]"]
---

# OOP in C++ -- Operator Overloading

*<- [[03_OOP_Static_vs_Dynamic_Polymorphism_CRTP|Static vs Dynamic & CRTP]] · [[03_OOP_Problems|Problems ->]]*

---

## 1. Member vs Non-Member Overloading Rules

Overloaded operators are functions with special signatures. To maintain standard language behavior, follow these structural rules:

### A. Operators That MUST Be Member Functions
The C++ standard forces the following operators to be non-static member functions of the class:
- **`operator=`** (Assignment)
- **`operator[]`** (Subscript access)
- **`operator()`** (Function call / Functor execution)
- **`operator->`** (Member selection pointer dereference)

---

### B. Symmetric Binary Operators (Use Friend Non-Members)
Symmetric operators (such as `+`, `-`, `*`, `/`, `==`) should be overloaded as **non-member friend functions**.
- **Reason**: Allowing implicit type conversions on **both** the left-hand and right-hand operands.
- If `operator+` is a member function, `myFraction + 2` compiles (implicit conversion of 2 via constructor), but `2 + myFraction` **fails** because the compiler cannot call a member operator on a primitive `int` literal.

```cpp
#include <iostream>

class Complex {
    double real;
    double imag;
public:
    Complex(double r, double i = 0.0) : real(r), imag(i) {}

    // Friend non-member: permits 'complexVal + 5.0' and '5.0 + complexVal'
    friend Complex operator+(const Complex& lhs, const Complex& rhs) {
        return Complex(lhs.real + rhs.real, lhs.imag + rhs.imag);
    }
};
```

---

## 2. Overloading Prefix vs Postfix Operators

Overloading increment (`++`) and decrement (`--`) operators requires distinguishing between their prefix and postfix forms.

### The Dummy Parameter Workaround
- **Prefix (`++obj`)**: Has signature `Type& operator++()`. It increments state and returns `*this` by reference (fast, no copies).
- **Postfix (`obj++`)**: Has signature `Type operator++(int)`. The `int` is a dummy compiler flag with no variable name. It saves a temporary copy of the old state, increments the actual object state, and returns the **old state by value** (slower due to temporary object construction).

```cpp
class Counter {
    int val = 0;
public:
    // Prefix: ++c
    Counter& operator++() {
        val++;          // Modify state
        return *this;   // Return by reference
    }

    // Postfix: c++
    Counter operator++(int) {
        Counter temp = *this; // Save copy of old state
        ++(*this);            // Call prefix operator to increment state
        return temp;          // Return old state by value
    }
};
```

> [!TIP]
> **Performance Tip**: In loop counters (e.g. `for (auto it = v.begin(); it != v.end(); ++it)`), always prefer prefix `++it` over postfix `it++`. For complex iterators, postfix creates and destroys a temporary iterator object on every iteration, wasting CPU cycles.

---

## 3. Subscript Operator (`operator[]`)

When overloading `operator[]` for collections, you must provide **two versions** to support both mutable and read-only const references.

```cpp
#include <stdexcept>

class IntegerList {
    int data[10] = {0};
public:
    // 1. Non-const: returns reference, allowing modifications
    int& operator[](size_t index) {
        if (index >= 10) throw std::out_of_range("Index out of bounds");
        return data[index];
    }

    // 2. Const: returns const reference / value for read-only access
    const int& operator[](size_t index) const {
        if (index >= 10) throw std::out_of_range("Index out of bounds");
        return data[index];
    }
};

void demoSubscript(const IntegerList& constList) {
    IntegerList list;
    list[0] = 42; // OK: calls non-const operator[]
    
    int val = constList[0]; // OK: calls const operator[]
    // constList[0] = 50;   // COMPILER ERROR: cannot modify through const reference!
}
```

---

## 4. The Spaceship Operator (`<=>`) & Ordering Categories

C++20 introduced the **Three-Way Comparison Operator (`<=>`)** to simplify comparisons.

### Default Member-wise Lexicographical Order:
Declaring `<=>` as `= default` auto-generates all six comparison operators (`==`, `!=`, `<`, `<=`, `>`, `>=`). It compares members sequentially in the order of their class declaration.

```cpp
#include <compare>

struct Employee {
    int rank;
    int age;

    // Automatically generates all 6 comparison operators!
    auto operator<=>(const Employee&) const = default;
};
```

### The Three Comparison Categories
The return type of `<=>` determines the ordering guarantees of the class:

1. **`std::strong_ordering`**:
   - Strict ordering. If `a <=> b` returns equivalent, they are identical and indistinguishable (e.g. integers, standard strings).
2. **`std::weak_ordering`**:
   - Equivalent values may have different details. E.g., case-insensitive string comparison: `"Hello" <=> "hello"` is equivalent, but they are not identical.
3. **`std::partial_ordering`**:
   - Some values cannot be compared. E.g., floating-point comparisons containing `NaN` (Not a Number). `NaN < 5.0` is false, and `NaN > 5.0` is also false.

---

*<- [[03_OOP_Static_vs_Dynamic_Polymorphism_CRTP|Static vs Dynamic & CRTP]] · [[03_OOP_Problems|Problems ->]]*
