---
tags: [cpp, templates, class-templates, function-templates, nttp]
links: ["[[04_Tmpl_Index]]", "[[04_Tmpl_Specialization]]"]
---

# Templates & Generics -- Function & Class Templates

*<- [[04_Tmpl_Index|Index]] · [[04_Tmpl_Specialization|Template Specialization ->]]*

---

## 1. Templates: Compile-Time Code Generators

In C++, templates are not compiled directly. Instead, they are blueprints. When you instantiate a template with concrete arguments, the compiler generates a custom version of the class or function for those types. This process is called **template instantiation**.

- **Template Argument Deduction**: The compiler automatically infers template parameters for functions based on argument types (no need to write `<Type>` explicitly).

```cpp
template <typename T>
T add(T a, T b) {
    return a + b;
}

void demoDeduction() {
    auto res1 = add(5, 10);     // Deduce T = int
    auto res2 = add(2.5, 3.5); // Deduce T = double
    // auto res3 = add(5, 2.5); // COMPILER ERROR: conflicting deduction for T (int vs double)
}
```

---

## 2. Class Templates

Class templates generate class structures parameterized by types.

```cpp
template <typename T>
class Box {
    T value;
public:
    Box(T val) : value(val) {}
    T getValue() const { return value; }
};
```

---

## 3. Non-Type Template Parameters (NTTP)

A template parameter can also be a **constant value** rather than a type. This is known as a **Non-Type Template Parameter (NTTP)**.

```cpp
#include <cstddef>
#include <stdexcept>

// NTTP 'Size' is bound at compile-time
template <typename T, std::size_t Size>
class StaticArray {
    T data[Size];
public:
    std::size_t getSize() const { return Size; }
    
    T& operator[](std::size_t index) {
        if (index >= Size) throw std::out_of_range("Index out of bounds!");
        return data[index];
    }
};

void demoNTTP() {
    StaticArray<int, 5> arr1; // Instantiates custom class of size 5
    StaticArray<int, 10> arr2; // Instantiates another distinct class of size 10
    
    // arr1 = arr2; // COMPILER ERROR: StaticArray<int, 5> and StaticArray<int, 10> are different classes!
}
```

### NTTP Standard Evolution:
- **Pre-C++20**: NTTPs were restricted to integral types (integers, bools, chars), pointers, and lvalue references.
- **C++20**: Relaxed to allow floating-point values (`double`, `float`) and certain literal struct types (with public members and defaulted equality operators) to be used as NTTPs.

---

*<- [[04_Tmpl_Index|Index]] · [[04_Tmpl_Specialization|Template Specialization ->]]*
