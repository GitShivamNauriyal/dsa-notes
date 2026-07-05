---
tags: [cpp, oop, constructors, destructors, raii, initialization-order, explicit-constructor, initializer-list]
links: ["[[03_OOP_Index]]", "[[03_OOP_Classes_and_Access_Control]]", "[[03_OOP_Rule_of_Zero_Three_Five]]"]
---

# OOP in C++ -- Constructors, Destructors, & RAII

*<- [[03_OOP_Classes_and_Access_Control|Classes & Access Control]] · [[03_OOP_Rule_of_Zero_Three_Five|Rule of Zero/Three/Five ->]]*

---

## 1. Resource Acquisition Is Initialization (RAII)

**RAII** is the core resource management pattern of C++. It links the lifecycle of a resource (heap memory, database locks, file handles) directly to the scope-bound lifetime of a stack object.

### The Stack Unwinding Guarantee:
When a block exits (either via regular return or when an exception is thrown), the C++ runtime automatically destroys all stack variables in reverse order of their construction. Because destructors of RAII wrapper classes execute automatically during this **stack unwinding** phase, resource leaks are completely eliminated.

```cpp
#include <mutex>
#include <stdexcept>

// [Original] Simple RAII Lock Wrapper Concept
class SimpleLock {
    std::mutex& mtx;
public:
    SimpleLock(std::mutex& m) : mtx(m) {
        mtx.lock(); // Acquire resource
    }
    ~SimpleLock() {
        mtx.unlock(); // Release resource
    }
    // Disable copy to prevent duplicate releases
    SimpleLock(const SimpleLock&) = delete;
    SimpleLock& operator=(const SimpleLock&) = delete;
};

std::mutex resourceMutex;
void safeAccess() {
    SimpleLock lock(resourceMutex); // Lock acquired
    
    // Critical Section
    if (someConditionFails()) {
        throw std::runtime_error("Error occurred!"); // Exception thrown
    }
    // lock goes out of scope here (or during exception unwinding); 
    // destructor runs and mutex is safely unlocked. No deadlocks!
}
```

---

## 2. Constructor Synthesis Rules

The compiler automatically synthesizes a **default constructor** `Widget()` only if **no other user-defined constructors** are declared.
- Declaring any parameterized constructor (e.g. `Widget(int)`) suppresses the synthesis of the default constructor.
- You can force the compiler to restore the default constructor using the `= default` specifier.

```cpp
class Widget {
public:
    Widget(int x) {}
    // Widget() = default; // If commented out, Widget w; will fail to compile!
};
```

---

## 3. Implicit Conversions & the `explicit` Keyword

Single-argument constructors can be used by the compiler to perform **implicit type conversions** to resolve argument matching. This can cause subtle bugs.

```cpp
class Buffer {
public:
    // Single-argument constructor
    Buffer(int capacity) {}
};

void printBuffer(const Buffer& buf) {}

void demoBuggyConversion() {
    // Intention: pass a Buffer object.
    // Reality: compiles fine! The compiler implicitly converts '42' into Buffer(42).
    printBuffer(42); 
}
```

### The Fix: `explicit` Constructors
Declaring a constructor as **`explicit`** prevents the compiler from performing implicit conversions or copy-initializations. The object must be initialized using direct initialization or explicit casts.

```cpp
class SafeBuffer {
public:
    explicit SafeBuffer(int capacity) {}
};

void demoSafe() {
    // SafeBuffer b = 10; // COMPILER ERROR: conversion is blocked!
    SafeBuffer b(10);     // OK: Direct initialization
}
```

> [!TIP]
> **Explicit Conversion Operators**: You can also mark conversion operators as `explicit` to prevent implicit evaluations, which is heavily used in standard types like `std::unique_ptr`'s `explicit operator bool()` to allow checks like `if (ptr)` but block `int x = ptr;`.

---

## 4. Initializer List Constructors (`std::initializer_list<T>`)

If a class defines a constructor accepting `std::initializer_list<T>`, it takes precedence over other constructors when using brace initialization `{}`.

```cpp
#include <vector>
#include <initializer_list>
#include <iostream>

class WidgetArray {
public:
    WidgetArray(int size, int defaultValue) {
        std::cout << "Normal constructor\n";
    }
    WidgetArray(std::initializer_list<int> list) {
        std::cout << "Initializer list constructor\n";
    }
};

void demoInitializer() {
    WidgetArray w1(10, 20); // Normal constructor called
    WidgetArray w2{10, 20}; // Initializer list constructor called!
}
```

---

## 5. Construction & Destruction Order in Inheritance

When creating a derived class object, memory is initialized in a strict hierarchical order.

### Order of Construction:
1. **Base Class Constructor**: Runs first (hierarchically down from root base).
2. **Derived Member Variables**: Initialized in the order of their declaration in the class definition.
3. **Derived Class Constructor Body**: Executes last.

### Order of Destruction:
Destruction happens in **exact reverse order**:
1. **Derived Class Destructor Body**: Executes first.
2. **Derived Member Variables**: Destroyed in reverse order of declaration.
3. **Base Class Destructor**: Executes last.

### ASCII Order Timeline

```
  Construction: ┌────────────────┐     ┌───────────────┐     ┌───────────────┐
                │ Base Construct ├────►│ Member Init   ├────►│ Derived Body  │
                └────────────────┘     └───────────────┘     └───────────────┘
                                                                     │
  Destruction:  ┌────────────────┐     ┌───────────────┐     ┌───────▼───────┐
                │ Base Destroy   │◄────┤ Member Destr  │◄────┤ Derived Body  │
                └────────────────┘     └───────────────┘     └───────────────┘
```

---

## 6. Member Initialization Order Trap

Remember that class members are initialized in their **declaration order inside the class definition**, not their order in the initializer list.

```cpp
class Trap {
    int a;
    int b; // 'a' is initialized before 'b'
public:
    // Trap: 'b' appears first in the list, but 'a' is initialized first!
    // Since 'a' reads uninitialized 'b', 'a' gets garbage values.
    Trap(int val) : b(val), a(b + 10) {} 
};
```

---

*<- [[03_OOP_Classes_and_Access_Control|Classes & Access Control]] · [[03_OOP_Rule_of_Zero_Three_Five|Rule of Zero/Three/Five ->]]*
