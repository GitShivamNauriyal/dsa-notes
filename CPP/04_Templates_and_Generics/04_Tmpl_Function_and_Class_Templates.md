---
tags: [cpp, templates, class-templates, function-templates, nttp, two-phase-lookup, compilation-model]
links: ["[[04_Tmpl_Index]]", "[[04_Tmpl_Specialization]]"]
---

# Templates & Generics -- Function & Class Templates

*<- [[04_Tmpl_Index|Index]] · [[04_Tmpl_Specialization|Template Specialization ->]]*

---

## 1. Templates: Compile-Time Instantiation

In C++, templates are not actual classes or functions. They are blueprints used by the compiler to generate code. When a template is called or instantiated, the compiler replaces the template parameters with concrete arguments and generates a distinct object code representation for those types. This process is called **template instantiation**.

### Two-Phase Lookup
Compilers parse templates in two distinct phases:
1. **Phase 1 (Parsing/Compilation Time)**: The compiler parses the template definition without knowing the template arguments. Only basic syntax validation is performed. Dependent names (names that depend on a template parameter) are not resolved.
2. **Phase 2 (Instantiation Time)**: When the template is instantiated with concrete types, the compiler does type resolution and validates whether the methods, variables, and operators used inside the template actually exist on those types.

```cpp
template <typename T>
void printProcessed(T obj) {
    // Phase 1: compiler checks syntax (brackets, semicolons).
    // Phase 2: compiler verifies if obj.serialize() exists and compiles for type T.
    obj.serialize(); 
}
```

---

## 2. Template Compilation Model

Because templates require Phase 2 instantiation, the compiler **must** see the entire definition (implementation) of a template in the translation unit where it is used.

### The Header Inclusion Rule
- If you declare a template in a header `Array.h` and implement its methods in `Array.cpp`, compiling `Array.cpp` succeeds, but compiling another file `main.cpp` that calls `Array<int>` will compile successfully but fail at **link-time** with an `undefined reference` error.
- **Why**: The compiler only instantiates the code for `Array<int>` when compiling `main.cpp`. But it doesn't have the implementation details from `Array.cpp` during that compile step, and `Array.cpp` was compiled without knowing that `int` was needed!
- **Fix**: Place the entire implementation of the template class directly in the header file.

### Explicit Instantiation (Compile-Time Optimization)
To avoid header bloat and reduce compilation times, you can keep the implementation in a `.cpp` file and explicitly instantiate the required types:

```cpp
// Array.h
template <typename T>
class Array {
    void process();
};

// Array.cpp
#include "Array.h"
template <typename T>
void Array<T>::process() { /* ... */ }

// Explicit Instantiation tells the compiler to generate code for 'int' now!
template class Array<int>;
```

---

## 3. Class and Function Templates

### Default Template Arguments
Classes and functions can specify default values for their template parameters.

```cpp
// Class template with default type and default NTTP value
template <typename T = int, int Capacity = 100>
class Stack {
    T data[Capacity];
};
```

---

## 4. Non-Type Template Parameters (NTTP)

An NTTP is a template parameter that represents a **constant value** rather than a type.

```cpp
#include <cstddef>
#include <iostream>

template <typename T, std::size_t Size>
class FixedArray {
    T data[Size];
public:
    constexpr std::size_t getSize() const { return Size; }
};

void demoNTTP() {
    FixedArray<int, 10> a;
    FixedArray<int, 20> b;
    // a = b; // COMPILER ERROR: FixedArray<int, 10> and FixedArray<int, 20> are distinct types!
}
```

### Evolution of NTTP Rules:
- **Pre-C++20**: NTTPs were restricted strictly to integral constants (integers, booleans, characters), pointers, and lvalue references.
- **C++20**: Relaxed restrictions to allow **floating-point types** (`double`, `float`) and **literal structural class types** (classes with only public members and defaulted equality operators) to serve as NTTPs.

---

*<- [[04_Tmpl_Index|Index]] · [[04_Tmpl_Specialization|Template Specialization ->]]*
