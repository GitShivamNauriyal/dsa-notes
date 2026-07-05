---
tags: [cpp, extras, sanitizers, asan, tsan, ubsan, msan, performance]
links: ["[[../../00_Home]]", "[[../../07_Concurrency/07_Conc_Index]]", "[[../../08_Concurrency_Primitives_From_Scratch/08_CPS_Index]]"]
---

# Extras -- LLVM/GCC Sanitizers

*<- [[../../00_Home|Home]]*

---

Sanitizers are compiler instrumentation tools that detect memory, concurrency, and behavior bugs at runtime with relatively low overhead compared to heavy VM simulators like Valgrind.

---

## 1. Sanitizer Suite Overview

| Sanitizer | Compile Flag | Detects | Runtime Overhead |
|---|---|---|---|
| **ASan** (AddressSanitizer) | `-fsanitize=address` | Use-after-free, heap/stack overflows, memory leaks | $2\times$ slowdown |
| **TSan** (ThreadSanitizer) | `-fsanitize=thread` | Data races, thread deadlocks | $5\times\text{--}10\times$ slowdown |
| **UBSan** (UndefinedBehaviorSanitizer) | `-fsanitize=undefined` | Signed integer overflow, division by zero, null dereferences | Minimal |
| **MSan** (MemorySanitizer) | `-fsanitize=memory` | Reads of uninitialized memory (Clang/Linux only) | $3\times$ slowdown |

> [!WARNING]
> You **cannot** combine `-fsanitize=address` and `-fsanitize=thread` in a single compilation run. They employ conflicting memory shadow mappings.

---

## 2. AddressSanitizer (ASan)

ASan maps 8 bytes of application memory to 1 byte of **Shadow Memory**, using instrumentation to track which bytes are safe to access (marked as unpoisoned) and which are illegal (poisoned, e.g. redzones around arrays).

```cpp
// Compile: g++ -fsanitize=address -g main.cpp -o main
void badAccess() {
    int* arr = new int[10];
    delete[] arr;
    
    // Use-After-Free detected instantly by ASan at runtime!
    arr[0] = 5; 
}
```

---

## 3. ThreadSanitizer (TSan)

TSan is critical when verifying the correctness of custom concurrency primitives (such as the spinlocks and queues implemented in `[[../../07_Concurrency/07_Conc_Index]]` and `[[../../08_Concurrency_Primitives_From_Scratch/08_CPS_Index]]`).
- **Data Race Definition**: Two threads access the same memory location concurrently, at least one access is a write, and there is no synchronization (mutex or atomic fence) ordering them.

```cpp
// Compile: g++ -fsanitize=thread -g main.cpp -o main
#include <thread>

int sharedData = 0;

void writer() { sharedData = 42; }
void reader() { int val = sharedData; }

void runRace() {
    std::thread t1(writer);
    std::thread t2(reader); // Data Race detected by TSan!
    t1.join();
    t2.join();
}
```

---

## 4. UndefinedBehaviorSanitizer (UBSan)

UBSan catches behaviors that the C++ standard does not define, which compilers might exploit for aggressive optimization (as detailed in `[[../../02_Core_Language_Mechanics/02_Core_Undefined_Behavior_Categories]]`).

```cpp
// Compile: g++ -fsanitize=undefined -g main.cpp -o main
void demoUB() {
    int maxVal = 2147483647;
    
    // Signed Overflow detected at runtime by UBSan!
    int overflow = maxVal + 1; 
}
```

---

*<- [[../../00_Home|Home]]*
