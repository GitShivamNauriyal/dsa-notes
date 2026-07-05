---
tags: [cpp, memory, problems, exercises]
links: ["[[06_Mem_Index]]", "[[06_Mem_Custom_Deleters_and_Allocators]]", "[[06_Mem_Solutions]]"]
---

# Memory & Smart Pointers -- Problems & Exercises

*<- [[06_Mem_Custom_Deleters_and_Allocators|Custom Deleters & Allocators]] · [[06_Mem_Solutions|Solutions ->]]*

---

## Tier 1 -- Concept Checks

### Question 1.1: Size of Smart Pointers
Given a class `Widget`, predict the output of `sizeof` for the following declarations on a 64-bit system:
1. `sizeof(Widget*)`
2. `sizeof(std::unique_ptr<Widget>)`
3. `sizeof(std::shared_ptr<Widget>)`
4. `sizeof(std::weak_ptr<Widget>)`

---

### Question 1.2: Reference Count Tracking
Trace the strong reference count of the managed object after each numbered statement in the following code block:
```cpp
auto p1 = std::make_shared<int>(10);  // (1)
auto p2 = p1;                        // (2)
std::weak_ptr<int> wp = p2;          // (3)
p1.reset();                          // (4)
auto p3 = wp.lock();                 // (5)
p2.reset();                          // (6)
p3.reset();                          // (7)
```

---

## Tier 2 -- Implementation

### Question 2.1: Cyclic References Memory Leak
Write classes `Parent` and `Child` where:
- `Parent` holds a `std::shared_ptr<Child>`.
- `Child` holds a `std::shared_ptr<Parent>`.
Demonstrate that instantiating and destroying these classes leaks memory (destructors are never called). Show how to fix the leak using `std::weak_ptr`.

---

## Tier 3 -- Interview-Level

### Question 3.1: Thread-Safe Object Cache using `weak_ptr`
In systems design, we want to share loaded objects (e.g., textures or database records) across multiple threads. If a thread requests an object that is already loaded, return it. If no threads are using the object, it should be automatically deleted from memory.
- Design a `TextureCache` class that uses `std::weak_ptr` and `std::shared_ptr` to achieve this.
- If multiple threads request the same texture name, only one `shared_ptr` allocation should exist. If all threads release the texture, the cache entry should expire.

---

## Tier 4 -- Systems / Placement-Hard

### Question 4.1: Custom `unique_ptr` Implementation
To verify deep understanding of ownership mechanics, write a custom implementation of unique pointer `my_unique_ptr<T, Deleter = std::default_delete<T>>` from scratch:
- Support custom deleters (defaulting to standard delete).
- Disable copy constructors and copy assignment operators.
- Support move constructor and move assignment operator.
- Support pointer operations: dereference `operator*`, member access `operator->`, and `get()`, `release()`, and `reset()`.
- Add static_assert checks to ensure pointer safety.

---

*<- [[06_Mem_Custom_Deleters_and_Allocators|Custom Deleters & Allocators]] · [[06_Mem_Solutions|Solutions ->]]*
