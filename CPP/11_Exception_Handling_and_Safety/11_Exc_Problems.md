---
tags: [cpp, exceptions, problems, exercises]
links: ["[[11_Exc_Index]]", "[[11_Exc_Stack_Unwinding_and_RAII]]", "[[11_Exc_Solutions]]"]
---

# Exception Handling & Safety -- Problems

*<- [[11_Exc_Stack_Unwinding_and_RAII|Stack Unwinding & RAII]] · [[11_Exc_Solutions|Solutions ->]]*

---

Solve the following exercises relating to exception safety guarantees and compile-time checks.

---

### Problem 1: Transactional Assignment (Strong Guarantee)
Write a class `TransactionalBuffer` that manages a heap-allocated array of integers.
- Implement the copy constructor, destructor, and a swap function.
- Implement the copy assignment operator (`operator=`) using the **Copy-and-Swap** idiom to satisfy the **Strong Exception Guarantee**.

---

### Problem 2: Audit `noexcept` Move Reallocations
Suppose you have a custom structure representing a network connection:

```cpp
struct Connection {
    int socketId;
    // ...
    Connection(Connection&& other) {
        socketId = other.socketId;
        other.socketId = -1;
    }
};
```

- **The Problem**: If you store instances of `Connection` in a `std::vector`, adding elements causes the vector to **copy** instead of **move** during capacity reallocations.
- **The Challenge**: Fix the `Connection` move constructor to enable move-based reallocations, and write a static assertion that verifies the optimization compiles at compile-time.

---

### Problem 3: Double Resource Allocation Safety
Write a function `processData` that must acquire two resource objects: `ResourceA` and `ResourceB` (both are allocated on the heap using raw `new` calls).
- If the allocation of `ResourceB` throws an exception, the function must guarantee that `ResourceA` is not leaked.
- Implement this **without** using smart pointers (use raw pointers and try-catch blocks to understand manual rollback mechanics).

---

*<- [[11_Exc_Stack_Unwinding_and_RAII|Stack Unwinding & RAII]] · [[11_Exc_Solutions|Solutions ->]]*
