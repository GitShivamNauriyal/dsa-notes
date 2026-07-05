---
tags: [cpp, core, linkage, storage-duration, static, extern, thread-local, odr]
links: ["[[02_Core_Index]]", "[[02_Core_Const_Constexpr_Consteval]]", "[[02_Core_Undefined_Behavior_Categories]]"]
---

# Core Language Mechanics -- Storage Duration & Linkage

*<- [[02_Core_Const_Constexpr_Consteval|Const & Compile-Time]] · [[02_Core_Undefined_Behavior_Categories|Undefined Behavior ->]]*

---

## 1. Storage Duration Categories

Storage duration determines the lifetime of an object's memory:

1. **Automatic Storage Duration**:
   - Allocated at the beginning of the enclosing code block and deallocated at the end (stack variables).
2. **Static Storage Duration**:
   - Allocated once at program startup and deallocated at program termination.
   - Declared at global/namespace scope, or using `static` inside functions/classes.
3. **Thread Storage Duration**:
   - Allocated when a thread starts and deallocated when the thread exits.
   - Declared using the **`thread_local`** keyword.
4. **Dynamic Storage Duration**:
   - Lifetime managed manually via heap allocation primitives (`new` / `delete`).

---

## 2. Linkage Types

Linkage determines whether names in one translation unit (TU) refer to the same entities in other TUs:

- **No Linkage**: Name can only be referenced from the scope it is declared in (e.g. local stack variables).
- **Internal Linkage**: Name is visible only within its own translation unit.
  - Declared using the **`static`** keyword at global scope, or inside an **anonymous namespace**.
- **External Linkage**: Name is visible and can be referenced from other TUs.
  - Non-const global variables and functions have external linkage by default.
  - Can be declared as shared using the **`extern`** keyword.

### Translation Unit & Linking Diagram

```
 Translation Unit A (file1.cpp)         Translation Unit B (file2.cpp)
 ┌───────────────────────────┐         ┌───────────────────────────┐
 │ static void helper();     │         │ extern int sharedCounter; │
 │ int sharedCounter = 42;   │         │ void process();           │
 └─────────────┬─────────────┘         └─────────────┬─────────────┘
               │ Compile                             │ Compile
               ▼                                     ▼
      file1.o Object File                   file2.o Object File
 ┌───────────────────────────┐         ┌───────────────────────────┐
 │ [Internal] helper         │         │ [Unresolved] sharedCounter│
 │ [Exported] sharedCounter  │         │ [Exported] process        │
 └─────────────┬─────────────┘         └─────────────┬─────────────┘
               │                                     │
               └──────────────────┬──────────────────┘
                                  │ Linker
                                  ▼
                            Executable/Lib
                (sharedCounter resolved to file1.o's address;
                 helper is invisible/unreachable by file2.o)
```

---

## 3. The One Definition Rule (ODR)

The **One Definition Rule (ODR)** states:
1. **Within a single Translation Unit**: An entity (variable, function, class, enum, template) can have at most one definition.
2. **Within the entire Program**:
   - Non-inline variables and functions can have **exactly one** definition.
   - Inline functions, inline variables, classes, and templates can be defined in multiple TUs, provided that each definition is identical token-for-token, and refers to the same entities.

### Inline Variables (C++17)
Prior to C++17, defining a global variable in a header file included by multiple source files violated ODR (duplicate symbols during linking). We had to declare it `extern` in the header and define it in exactly one source file.
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
