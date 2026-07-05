---
tags: [cpp, oop, rule-of-three, rule-of-five, rule-of-zero, special-member-functions, copy-and-swap, generation-rules]
links: ["[[03_OOP_Index]]", "[[03_OOP_Constructors_Destructors_and_RAII]]", "[[03_OOP_Inheritance_and_Virtual_Dispatch]]"]
---

# OOP in C++ -- Rule of Zero, Three, & Five

*<- [[03_OOP_Constructors_Destructors_and_RAII|Constructors & RAII]] · [[03_OOP_Inheritance_and_Virtual_Dispatch|Inheritance & Virtual Dispatch ->]]*

---

## 1. Compiler Generation Rules for Special Member Functions

A critical topic in C++ compiler theory is when the compiler automatically generates (synthesizes) special member functions, and when it suppresses them.

### The Special Member Functions Generation Matrix:
If you declare *any* of the functions in the left column, the compiler's automatic generation behavior changes for the other functions as follows:

| User Declared | Default Constructor | Destructor | Copy Constructor | Copy Assignment | Move Constructor | Move Assignment |
|---|---|---|---|---|---|---|
| **Nothing** | Generated | Generated | Generated | Generated | Generated | Generated |
| **Constructor (any)**| **Suppressed** | Generated | Generated | Generated | Generated | Generated |
| **Destructor** | Generated | User Defined | Generated (deprecated)| Generated (deprecated)| **Suppressed** | **Suppressed** |
| **Copy Constructor** | **Suppressed** | Generated | User Defined | Generated | **Suppressed** | **Suppressed** |
| **Copy Assignment** | Generated | Generated | Generated | User Defined | **Suppressed** | **Suppressed** |
| **Move Constructor** | **Suppressed** | Generated | **Deleted** | **Deleted** | User Defined | **Suppressed** |
| **Move Assignment** | Generated | Generated | **Deleted** | **Deleted** | **Suppressed** | User Defined |

> [!IMPORTANT]
> - Declaring a custom **destructor** suppresses the automatic generation of move operations.
> - Declaring any **move operation** automatically deletes the copy operations to prevent accidental shallow copies of resource handles.

---

## 2. Rule of Three (Pre-C++11 Resource Management)

If a class manages raw resources (e.g. raw pointer allocations, system file descriptors), you must manually release that resource. Therefore, you must write a custom **Destructor**.
If you write a destructor, the compiler-generated copy constructor performs a member-wise shallow copy. If one object is copied, both point to the same memory. When they go out of scope, both call `delete` on the same pointer, causing a **double-free crash**.

To prevent this, you must write:
1. **Destructor**: To free raw memory.
2. **Copy Constructor**: To allocate separate memory and copy contents (Deep Copy).
3. **Copy Assignment Operator**: To free old memory, allocate new memory, and copy contents.

---

## 3. Rule of Five (Modern C++ Resource Management)

In C++11, move semantics were introduced. If your class manages resources, copy operations are expensive. You should implement move operations to allow resources to be "stolen" from temporary objects without deep copying.

### The Copy-and-Swap Idiom (The Assignment Gold Standard)
The traditional way of writing a copy assignment operator requires checking for self-assignment `if (this != &other)` and manually deleting old data. If memory allocation for the new copy fails, the destination object is left partially deallocated and corrupted.

The **Copy-and-Swap Idiom** solves this by:
1. Passing the argument **by value** to trigger the copy constructor automatically. If allocation fails, the exception is thrown before entering the assignment function body, leaving the destination object safe.
2. Swapping the contents of `this` with the temporary copy.
3. The temporary copy goes out of scope and automatically deallocates the old resource.
- **Benefit**: Provides the **Strong Exception Safety Guarantee** and prevents self-assignment bugs naturally.

```cpp
#include <iostream>
#include <algorithm>

class ManagedIntArray {
    int* data;
    size_t size;

public:
    explicit ManagedIntArray(size_t sz) : size(sz), data(new int[sz]()) {}

    // 1. Destructor
    ~ManagedIntArray() {
        delete[] data;
    }

    // 2. Copy Constructor (Deep Copy)
    ManagedIntArray(const ManagedIntArray& other) : size(other.size), data(new int[other.size]) {
        std::copy(other.data, other.data + size, data);
    }

    // Friend swap helper
    friend void swap(ManagedIntArray& first, ManagedIntArray& second) noexcept {
        using std::swap;
        swap(first.data, second.data);
        swap(first.size, second.size);
    }

    // 3. Copy Assignment Operator using Copy-and-Swap
    // Argument passed by value (creates a temporary copy)
    ManagedIntArray& operator=(ManagedIntArray other) {
        swap(*this, other); // Swap internal states
        return *this;       // Old state is destroyed when 'other' goes out of scope
    }

    // 4. Move Constructor (Resource theft)
    ManagedIntArray(ManagedIntArray&& other) noexcept : data(nullptr), size(0) {
        swap(*this, other);
    }

    // 5. Move Assignment Operator
    ManagedIntArray& operator=(ManagedIntArray&& other) noexcept {
        ManagedIntArray temp(std::move(other));
        swap(*this, temp);
        return *this;
    }
};
```

---

## 4. Rule of Zero

The **Rule of Zero** states that your custom application classes should never declare any of the five special member functions. Instead, delegate resource management to standard library components (`std::unique_ptr`, `std::string`, `std::vector`) which already implement the Rule of Five correctly.

```cpp
#include <string>
#include <vector>
#include <memory>

// Zero custom boilerplate! Compiler generates all 5 functions automatically and safely.
class Employee {
    std::string name;
    std::vector<double> baseSalaries;
    std::unique_ptr<int> idCode;
};
```

---

## 5. Trivial vs Non-Trivial Types

In systems engineering, the compiler optimizes copy operations if a class is **trivially copyable**:
- **Trivial Type**: A class that has a default constructor, and no virtual functions or virtual base classes.
- **Trivially Copyable**: A class that can be copied by copying its raw bytes directly (e.g. using `std::memcpy`). It has no custom copy/move constructors or assignment operators.
- You can query this property at compile-time: `std::is_trivially_copyable_v<T>`.

```cpp
#include <type_traits>

struct PlainData {
    int x;
    double y;
}; // Trivially copyable

class NonTrivial {
    std::string s; // std::string has custom copy constructor -> non-trivial!
};

void checkTrivial() {
    static_assert(std::is_trivially_copyable_v<PlainData>, "PlainData must be trivially copyable!");
    // static_assert(std::is_trivially_copyable_v<NonTrivial>, "Fails!");
}
```

---

*<- [[03_OOP_Constructors_Destructors_and_RAII|Constructors & RAII]] · [[03_OOP_Inheritance_and_Virtual_Dispatch|Inheritance & Virtual Dispatch ->]]*
