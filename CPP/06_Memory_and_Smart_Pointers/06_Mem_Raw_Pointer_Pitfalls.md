---
tags: [cpp, memory, raw-pointers, memory-leaks, double-free]
links: ["[[06_Mem_Index]]", "[[06_Mem_Unique_Ptr]]"]
---

# Memory & Smart Pointers -- Raw Pointer Pitfalls

*<- [[06_Mem_Index|Index]] · [[06_Mem_Unique_Ptr|Unique Pointers ->]]*

---

## The Dangers of Manual Memory Management

In C++ (prior to smart pointers), managing heap memory manually using `new` and `delete` was highly error-prone. The primary issues include:

---

## 1. Memory Leaks (Exceptions Bypass)
If a function allocates memory on the heap and an exception is thrown before `delete` is reached, the memory is leaked.

```cpp
void leakDemo() {
    int* data = new int[100]; // Allocate memory
    
    // If this function throws an exception, delete is never called.
    // 'data' pointer on stack is cleaned up, but heap allocation stays leaked!
    doSomething(); 
    
    delete[] data;
}
```

---

## 2. Double Free Corruption
Calling `delete` on the same pointer more than once. This corrupts the internal bucket tables of the heap allocator, leading to security vulnerabilities or immediate crashes.

```cpp
void doubleFreeDemo() {
    int* ptr = new int(42);
    
    delete ptr; // Free memory
    // ...
    delete ptr; // UB: Double free!
}
```

---

## 3. Dangling Pointers
Dereferencing a pointer after the memory it refers to has been deallocated.

```cpp
void danglingDemo() {
    int* ptr = new int(100);
    delete ptr; // Memory freed
    
    // ptr is now a dangling pointer (still holds the address of deallocated memory)
    *ptr = 50; // UB: Writing to deallocated memory!
}
```

---

## The Modern C++ Solution (RAII / Smart Pointers)
Instead of managing raw pointers, wrap heap allocations inside smart pointer classes (`std::unique_ptr`, `std::shared_ptr`). Their destructors automatically call `delete` when the pointer object goes out of scope on the stack, ensuring absolute safety even during exception unwinding.

---

*<- [[06_Mem_Index|Index]] · [[06_Mem_Unique_Ptr|Unique Pointers ->]]*
