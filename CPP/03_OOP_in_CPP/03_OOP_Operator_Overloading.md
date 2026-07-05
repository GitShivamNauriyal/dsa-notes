---
tags: [cpp, oop, operator-overloading, spaceship-operator, comparison]
links: ["[[03_OOP_Index]]", "[[03_OOP_Static_vs_Dynamic_Polymorphism_CRTP]]", "[[03_OOP_Problems]]"]
---

# OOP in C++ -- Operator Overloading

*<- [[03_OOP_Static_vs_Dynamic_Polymorphism_CRTP|Static vs Dynamic & CRTP]] · [[03_OOP_Problems|Problems ->]]*

---

## 1. Member vs Non-Member Overloading Rules

When overloading operators, follow these design principles:

1. **Member Operators**:
   - Use for operators that modify the left-hand object: `+=`, `-=`, `=`, `++`, `--`.
   - Must be members for: `[]`, `()`, `->`, `=`.
2. **Non-Member (Friend) Operators**:
   - Use for symmetric binary operators to allow implicit type conversion on both operands: `+`, `-`, `*`, `/`, `==`, `!=`.
   - Mandatory for stream operators `<<` and `>>` because the left-hand operand is an stream object (`std::ostream` / `std::istream`), which you cannot modify.

```cpp
#include <iostream>

class Fraction {
    int num;
    int den;
public:
    Fraction(int n, int d = 1) : num(n), den(d) {}

    // Member += operator (modifies left-hand side)
    Fraction& operator+=(const Fraction& other) {
        num = num * other.den + other.num * den;
        den = den * other.den;
        return *this;
    }

    // Friend non-member operator (symmetric, allows: 2 + frac and frac + 2)
    friend Fraction operator+(Fraction lhs, const Fraction& rhs) {
        lhs += rhs;
        return lhs;
    }

    // Friend stream insertion operator
    friend std::ostream& operator<<(std::ostream& os, const Fraction& f) {
        os << f.num << "/" << f.den;
        return os;
    }
};
```

---

## 2. C++20 Spaceship Operator (`<=>`)

Prior to C++20, to make a custom class fully comparable, we had to write six separate comparison functions (`==`, `!=`, `<`, `>`, `<=`, `>=`).
C++20 introduced the **Three-Way Comparison Operator (`<=>`)**, also known as the **spaceship operator**.

### Default Lexicographical Comparisons:
By defining `<=>` and `==` as `= default`, the compiler automatically generates all six comparison operators for you! It compares members lexicographically in the order they are declared in the class.

```cpp
#include <compare>
#include <iostream>

class Point {
    int x;
    int y;
public:
    Point(int xVal, int yVal) : x(xVal), y(yVal) {}

    // Automatically generates all 6 comparison operators!
    auto operator<=>(const Point&) const = default;
};

void demoSpaceship() {
    Point p1(1, 2);
    Point p2(1, 3);

    if (p1 < p2) {
        std::cout << "p1 is less than p2\n"; // Executes: compares x first, then y.
    }
}
```

### Comparison Category Return Types:
The spaceship operator returns one of three comparison categories depending on the member types:
1. **`std::strong_ordering`**: Strict ordering (elements are either less, greater, or equivalent/equal, e.g., integers).
2. **`std::weak_ordering`**: Equivalent values might not be identical (e.g. case-insensitive string match).
3. **`std::partial_ordering`**: Some values are incomparable (e.g. float `NaN` comparisons).

---

*<- [[03_OOP_Static_vs_Dynamic_Polymorphism_CRTP|Static vs Dynamic & CRTP]] · [[03_OOP_Problems|Problems ->]]*
