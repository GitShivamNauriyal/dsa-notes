---
tags: [cpp, stl, lambdas, init-capture, mutable, recursive-lambdas, closure-class]
links: ["[[05_STL_Index]]", "[[05_STL_Algorithms_Library]]"]
---

# STL & Modern Features -- Lambdas Deep Dive

*<- [[05_STL_Index|Index]] · [[05_STL_Algorithms_Library|Algorithms Library ->]]*

---

## 1. Closure Class Internals

A lambda expression is compile-time syntactic sugar for generating a **functor** (an object of a unique, unnamed class type with an overloaded call operator `operator()`). This unnamed class is called the **closure class**.

### Under the Hood Code Translation
When you write:
```cpp
int x = 10;
int y = 20;
auto myLambda = [x, &y](int val) {
    return x + y + val;
};
```

The compiler translates it into a structure resembling this:
```cpp
class UnnamedClosureClass {
    int x;     // Captured by value
    int& y;    // Captured by reference
public:
    // Constructor initializes captured states
    UnnamedClosureClass(int _x, int& _y) : x(_x), y(_y) {}

    // Call operator is const by default
    auto operator()(int val) const {
        return x + y + val;
    }

    // Disable assignment operator
    UnnamedClosureClass& operator=(const UnnamedClosureClass&) = delete;
};

// Instantiation:
UnnamedClosureClass myLambda(x, y);
```

---

## 2. Capture Clause Mechanics

### A. Stateless Lambdas to Function Pointer Conversion
If a lambda has an empty capture list `[]`, the compiler generates an implicit conversion operator to a **raw C-style function pointer**. This is useful for passing lambdas to legacy C APIs.

```cpp
void executeCallback(void (*fp)(int)) {
    fp(42);
}

void demoStateless() {
    // Stateless lambda converts implicitly to void(*)(int)
    executeCallback([](int x) {
        std::cout << "Value: " << x << "\n";
    });
}
```

### B. Init-Capture / Move-Capture (C++14)
Allows declaring and initializing variables inside the capture clause. This is the only way to capture move-only objects (like `std::unique_ptr`).

```cpp
#include <memory>

void demoInitCapture() {
    auto uptr = std::make_unique<int>(100);

    // Capture by moving ownership
    auto task = [ptr = std::move(uptr)]() {
        return *ptr + 10;
    };
}
```

### C. Class Member Capture (`this` vs `*this`)
- `[this]`: Captures the class pointer by value. If the enclosing class object is destroyed, calling the lambda dereferences a dangling pointer (classic callback crash).
- `[*this]` *(C++17)*: Captures a **copy of the entire object**, securing data safety.

---

## 3. The `mutable` Keyword

By default, the closure class's `operator()` is declared `const`, meaning members captured by value cannot be modified. Declaring the lambda **`mutable`** strips this `const` restriction.

```cpp
void demoMutable() {
    int count = 0;
    
    // 'mutable' allows modifying the internal copy of 'count'
    auto increment = [count]() mutable {
        count++; // Allowed: modifies internal member variable UnnamedClosureClass::count
    };
}
```

---

## 4. `constexpr` & `static` Lambdas (C++17 / C++23)

- **`constexpr` Lambdas** (C++17): A lambda is implicitly `constexpr` if its body can be evaluated at compile-time. You can also explicitly mark it: `[]() constexpr {}`.
- **`static` Lambdas** (C++23): If a lambda does not capture any variables, it can be declared `static`. This optimizes performance by removing the need for a closure instance object entirely.

```cpp
void demoModernLambdas() {
    // C++17 constexpr lambda
    auto square = [](int x) constexpr { return x * x; };
    static_assert(square(5) == 25, "Must compile-time evaluate");

    // C++23 static lambda (no capture allowed)
    auto log = static [](int x) { std::cout << x << "\n"; };
}
```

---

## 5. C++23 Recursive Lambdas (Deducing `this`)

C++23 introduced **Deducing `this`** (explicit object parameters), allowing a lambda to receive a reference to its own closure object as its first argument.

```cpp
#include <iostream>

void demoRecursion() {
    // C++23: 'this auto self' allows calling itself directly
    auto fib = [](this auto self, int n) -> int {
        if (n <= 1) return n;
        return self(n - 1) + self(n - 2);
    };

    std::cout << fib(6) << "\n"; // Outputs 8
}
```

---

*<- [[05_STL_Index|Index]] · [[05_STL_Algorithms_Library|Algorithms Library ->]]*
