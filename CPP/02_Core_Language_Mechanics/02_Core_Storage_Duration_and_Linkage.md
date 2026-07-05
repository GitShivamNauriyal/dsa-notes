---
tags: [cpp, core, linkage, storage-duration, static, extern, thread-local, anonymous-namespaces, odr]
links: ["[[02_Core_Index]]", "[[02_Core_Const_Constexpr_Consteval]]", "[[02_Core_Undefined_Behavior_Categories]]"]
---

# Core Language Mechanics -- Storage Duration & Linkage

*<- [[02_Core_Const_Constexpr_Consteval|Const & Compile-Time]] · [[02_Core_Undefined_Behavior_Categories|Undefined Behavior ->]]*

---

## 1. Storage Duration Categories

Storage duration determines the lifetime and deallocation timeline of an object's memory allocation:

1. **Automatic Storage Duration**:
   - Allocated on the stack when entering the enclosing block and destroyed when the block exits.
2. **Static Storage Duration**:
   - Allocated once at program startup (initialized either at compile-time or the first time thread execution hits the definition for local statics) and destroyed at program shutdown.
3. **Thread Storage Duration (C++11)**:
   - Separate instance allocated for each thread. Created when the thread starts and destroyed when the thread exits. Exposes the **`thread_local`** keyword.
4. **Dynamic Storage Duration**:
   - Lifetime managed manually via heap allocation primitives (`new` / `delete`).

```cpp
#include <iostream>
#include <thread>

// Thread-local variable: initialized once per thread
thread_local int threadCounter = 0; 

void work() {
    threadCounter++;
    std::cout << "Thread " << std::this_thread::get_id() << " counter: " << threadCounter << "\n";
}

void demoThreadLocal() {
    std::thread t1(work); // Prints: Thread ... counter: 1
    std::thread t2(work); // Prints: Thread ... counter: 1 (separate variable!)
    t1.join();
    t2.join();
}
```

---

## 2. Linkage Types

Linkage determines whether a name (variable, function, type) declared in one translation unit (TU) refers to the same name in another translation unit.

- **No Linkage**: Name can only be referenced from the scope it is declared in (e.g. local stack variables).
- **Internal Linkage**: Name is visible only within its own translation unit.
  - Declared using the **`static`** keyword at global scope, or inside an **anonymous namespace**.
- **External Linkage**: Name is visible and can be referenced from other TUs.
  - Declared by default for non-const global variables and functions. Referenced using the **`extern`** keyword.

---

### Static Globals vs Anonymous Namespaces
In modern C++, **anonymous namespaces** are preferred over the `static` keyword for declaring internal linkage.

- **`static` Limitation**: The `static` keyword can only be applied to variables and functions. It cannot be applied to class/struct declarations or type aliases.
- **Anonymous Namespaces**: Automatically give internal linkage to *everything* declared inside them, including classes, structures, type aliases, variables, and functions.

```cpp
namespace {
    // Both class Helper and variable localState have internal linkage!
    // They are invisible and unreachable outside this translation unit.
    class Helper {
    public:
        void run() {}
    };

    int localState = 100;
}
```

---

## 3. The One Definition Rule (ODR) & Inline Variables

The **One Definition Rule (ODR)** states:
1. **Within a single Translation Unit**: An entity (variable, function, class, template) can have at most one definition.
2. **Within the entire Program**:
   - Non-inline variables and functions can have **exactly one** definition across the entire codebase.
   - Inline functions, inline variables, classes, and templates can be defined in multiple TUs, provided that each definition is identical token-for-token, and refers to the same entities.

### Inline Variables (C++17)
Before C++17, defining a global variable in a header file included by multiple source files violated ODR (duplicate symbols during linking). We had to declare it `extern` in the header and define it in exactly one source file.
From C++17, declaring a variable **`inline`** allows it to be defined in multiple TUs (via header inclusion) without violating ODR. The linker collapses all copies into a single global instance.

```cpp
// global_config.h
#pragma once
#include <string>

// C++17 Inline variable: safe to include in multiple .cpp files!
inline std::string globalAppName = "TradingEngine";
```

---

*<- [[02_Core_Const_Constexpr_Consteval|Const & Compile-Time]] · [[02_Core_Undefined_Behavior_Categories|Undefined Behavior ->]]*
