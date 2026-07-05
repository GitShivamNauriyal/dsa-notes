---
tags: [cpp, stl, lambdas, init-capture, mutable, recursive-lambdas]
links: ["[[05_STL_Index]]", "[[05_STL_Algorithms_Library]]"]
---

# STL & Modern Features -- Lambdas Deep Dive

*<- [[05_STL_Index|Index]] · [[05_STL_Algorithms_Library|Algorithms Library ->]]*

---

## 1. Lambda Closure Internals

A lambda expression is syntactic sugar for a **functor** (an object of an anonymous class type with an overloaded call operator `operator()`).
- The compiler translates variables in the capture clause into member variables of this anonymous class.

---

## 2. Capture Clause Mechanics

### A. Value vs Reference Captures
- `[=]`: Capture all referenced local variables by value (read-only by default).
- `[&]`: Capture all referenced local variables by reference.

### B. Init-Capture / Move-Capture (C++14)
Allows you to create and initialize new variables in the capture clause. This is vital for capturing move-only types like `std::unique_ptr`.

```cpp
#include <memory>
#include <utility>

void demoInitCapture() {
    auto ptr = std::make_unique<int>(100);
    
    // C++14 Init-capture: move ptr into the lambda closure
    auto lambda = [myPtr = std::move(ptr)]() {
        return *myPtr + 5;
    };
}
```

### C. Capturing Class Members (`this` vs `*this`)
- `[this]`: Captures the `this` pointer by value. **Dangerous**: If the enclosing object is destroyed, calling the lambda dereferences a dangling pointer!
- `[*this]` *(C++17)*: Captures a **copy** of the entire object, keeping the state safe even if the original object dies.

---

## 3. The `mutable` Keyword

By default, the compiler-generated `operator()` of a lambda is a `const` member function. This means variables captured by value cannot be modified inside the lambda body. Prepending **`mutable`** removes this `const` restriction.

```cpp
void demoMutable() {
    int counter = 0;
    
    // Prepending 'mutable' allows modifying captured value 'counter'
    auto increment = [counter]() mutable {
        counter++; // Modifies the internal copy of counter inside the closure
    };
    
    increment();
    // Local 'counter' remains 0 because it was captured by value!
}
```

---

## 4. C++23 Recursive Lambdas (Deducing `this`)

Prior to C++23, making a lambda recursive was awkward (requiring wrapping in `std::function` which adds runtime overhead, or passing the lambda as a parameter to itself).
C++23 introduced **Deducing `this`**, allowing a lambda to accept its own closure object as the first parameter to enable direct recursion.

```cpp
#include <iostream>

void demoRecursiveLambda() {
    // C++23: 'this auto self' allows the lambda to refer to itself
    auto factorial = [](this auto self, int n) -> int {
        if (n <= 1) return 1;
        return n * self(n - 1); // Recurse!
    };
    
    std::cout << factorial(5) << "\n"; // Prints 120
}
```

---

*<- [[05_STL_Index|Index]] · [[05_STL_Algorithms_Library|Algorithms Library ->]]*
