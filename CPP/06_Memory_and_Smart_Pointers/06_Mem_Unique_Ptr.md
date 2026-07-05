---
tags: [cpp, memory, unique-ptr, exclusive-ownership, make-unique, exception-safety]
links: ["[[06_Mem_Index]]", "[[06_Mem_Raw_Pointer_Pitfalls]]", "[[06_Mem_Shared_Ptr_and_Weak_Ptr]]"]
---

# Memory & Smart Pointers -- Unique Pointers

*<- [[06_Mem_Raw_Pointer_Pitfalls|Raw Pointer Pitfalls]] · [[06_Mem_Shared_Ptr_and_Weak_Ptr|Shared & Weak Pointers ->]]*

---

## 1. Exclusive Ownership Model

`std::unique_ptr` manages a heap object with **exclusive ownership**.
- Only one `std::unique_ptr` can own the object at a time.
- **Non-Copyable**: Copy constructor and copy assignment are `= delete`d.
- **Movable**: Ownership can be transferred using `std::move`.

```cpp
#include <memory>
#include <utility>

void demoUnique() {
    auto p1 = std::make_unique<int>(10);
    // std::unique_ptr<int> p2 = p1; // COMPILER ERROR: Copy is deleted!
    
    std::unique_ptr<int> p2 = std::move(p1); // Transfer ownership. p1 is now nullptr
}
```

---

## 2. Exception Safety & `std::make_unique` (C++14)

Prior to C++14, you had to allocate memory using raw `new` and wrap it:
`std::unique_ptr<Widget>(new Widget())`.

### The Sequence Trap (Memory Leak)
If a function call contains multiple arguments, the C++ standard does not specify the evaluation order of those arguments.

```cpp
void execute(std::unique_ptr<Widget> ptr, int val);

// Calling:
execute(std::unique_ptr<Widget>(new Widget()), throwException());
```

If the compiler executes:
1. `new Widget()` (allocates heap memory).
2. `throwException()` (throws an exception).
3. `std::unique_ptr<Widget>` constructor.
- Because the exception is thrown *before* the unique_ptr constructor runs and takes ownership, the allocated `Widget` memory is **permanently leaked**!

### The Solution: `std::make_unique`
By executing allocation and construction inside a single function, the operation is atomic with respect to outside parameters. If an exception occurs, cleanup is guaranteed.

```cpp
execute(std::make_unique<Widget>(), throwException()); // Safe: no leaks possible!
```

---

## 3. Size Overhead & Custom Deleters

A default `std::unique_ptr` occupies exactly the same memory size as a raw pointer (`sizeof(std::unique_ptr<T>) == sizeof(T*)` - 8 bytes). However, supplying custom deleters can alter this.

### Size Overhead Matrix:
- **Stateless Functor / Empty Class**: **8 bytes**. The compiler applies the **Empty Base Class Optimization (EBCO)** to optimize away the deleter's size.
- **Function Pointer**: **16 bytes**. The unique_ptr must store the raw pointer (8 bytes) + the function pointer (8 bytes).
- **Stateful Functor / Capture Lambda**: **16+ bytes** depending on the variables captured.

```cpp
#include <memory>
#include <iostream>

// 1. Stateless Functor (EBCO optimizes size to 8 bytes)
struct StatelessDeleter {
    void operator()(int* p) const { delete p; }
};

// 2. Function Pointer Deleter (Increases size to 16 bytes)
void functionDeleter(int* p) { delete p; }

void demoSizes() {
    std::unique_ptr<int> pDefault;
    std::unique_ptr<int, StatelessDeleter> pStateless;
    std::unique_ptr<int, void(*)(int*)> pFuncPtr(new int(10), functionDeleter);

    std::cout << sizeof(pDefault) << "\n";   // Outputs 8
    std::cout << sizeof(pStateless) << "\n"; // Outputs 8 (EBCO applied!)
    std::cout << sizeof(pFuncPtr) << "\n";   // Outputs 16
}
```

---

*<- [[06_Mem_Raw_Pointer_Pitfalls|Raw Pointer Pitfalls]] · [[06_Mem_Shared_Ptr_and_Weak_Ptr|Shared & Weak Pointers ->]]*
