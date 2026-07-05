---
tags: [cpp, memory, shared-ptr, weak-ptr, control-block, cyclic-references]
links: ["[[06_Mem_Index]]", "[[06_Mem_Unique_Ptr]]", "[[06_Mem_Move_Semantics_and_Perfect_Forwarding]]"]
---

# Memory & Smart Pointers -- Shared & Weak Pointers

*<- [[06_Mem_Unique_Ptr|Unique Pointers]] · [[06_Mem_Move_Semantics_and_Perfect_Forwarding|Move & Perfect Forwarding ->]]*

---

## 1. Reference Counting & Shared Ownership

`std::shared_ptr` implements **shared ownership**. 
- Multiple `std::shared_ptr` instances can point to the same object.
- The object is deleted only when the last `std::shared_ptr` pointing to it is destroyed.
- **Reference Count**: Tracks how many `shared_ptr` instances exist. Incremented on copy, decremented on destruction.

---

## 2. Control Block Memory Layout

Under the hood, a `std::shared_ptr` consists of **two pointers** on the stack:
1. A pointer to the managed object.
2. A pointer to a heap-allocated **Control Block**.

### ASCII Memory Layout

```
    Stack                                       Heap
  ┌─────────────────┐                        ┌──────────────────┐
  │ ptr to Object  ─┼───────────────────────►│  Managed Object  │
  ├─────────────────┤                        └──────────────────┘
  │ ptr to Ctrl Blk ┼──────────┐                      ▲
  └─────────────────┘          │             ┌────────┴─────────┐
                               └────────────►│  Control Block:  │
                                             │  Strong Count    │
                                             │  Weak Count      │
                                             │  Custom Deleter  │
                                             └──────────────────┘
```

The **Control Block** contains:
- **Strong Reference Count**: Number of `std::shared_ptr` instances owning the object.
- **Weak Reference Count**: Number of `std::weak_ptr` instances observing the object.
- Custom deleters or allocators, if provided.

- **Size**: A `std::shared_ptr` is double the size of a raw pointer (`16` bytes on 64-bit systems).

---

## 3. `std::weak_ptr` (The Non-Owning Observer)

`std::weak_ptr` points to an object managed by `std::shared_ptr` but **does not increment the strong reference count**.
- It is used to observe an object without claiming ownership.
- To access the object, you must call **`lock()`**, which returns a `std::shared_ptr` (or `nullptr` if the object has already been destroyed).

```cpp
#include <memory>
#include <iostream>

void demoWeakPtr() {
    auto shared = std::make_shared<int>(42);
    std::weak_ptr<int> weak = shared; // weak observes shared

    // To read the value: lock() checks if the object is still alive
    if (auto lockedShared = weak.lock()) {
        std::cout << *lockedShared << "\n"; // Safe: lockedShared owns it during this scope
    }

    shared.reset(); // destroy original object

    if (weak.expired()) {
        std::cout << "Object is gone!\n"; // Executes
    }
}
```

---

## 4. Breaking Cyclic References

If two objects contain `std::shared_ptr` pointing to each other, their reference counts can never drop to 0, causing a permanent **memory leak**. Replacing one of the pointers with `std::weak_ptr` breaks the cycle.

```cpp
#include <memory>

struct Node {
    // std::shared_ptr<Node> next; // If both use shared_ptr, cycle forms!
    std::weak_ptr<Node> prev;      // Use weak_ptr here to break the cycle
    std::shared_ptr<Node> next;
};
```

---

*<- [[06_Mem_Unique_Ptr|Unique Pointers]] · [[06_Mem_Move_Semantics_and_Perfect_Forwarding|Move & Perfect Forwarding ->]]*
