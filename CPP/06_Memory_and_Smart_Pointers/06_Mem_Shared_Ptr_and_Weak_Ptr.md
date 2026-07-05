---
tags: [cpp, memory, shared-ptr, weak-ptr, control-block, cyclic-references, thread-safety]
links: ["[[06_Mem_Index]]", "[[06_Mem_Unique_Ptr]]", "[[06_Mem_Move_Semantics_and_Perfect_Forwarding]]"]
---

# Memory & Smart Pointers -- Shared & Weak Pointers

*<- [[06_Mem_Unique_Ptr|Unique Pointers]] · [[06_Mem_Move_Semantics_and_Perfect_Forwarding|Move & Perfect Forwarding ->]]*

---

## 1. Reference Counting & Control Block

`std::shared_ptr` implements **shared ownership**. Multiple pointers can own the same resource, and the resource is deleted only when the last owner releases it.

### Stack and Heap Memory Layout
Under the hood, a `std::shared_ptr` consists of **two pointers** stored on the stack:
1. A pointer to the managed object.
2. A pointer to the heap-allocated **Control Block**.

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

- **Control Block**: Holds the strong reference count, the weak reference count, and any custom deleter or allocator.
- **Allocation Cost**: 
  - Using `std::shared_ptr<T>(new T)` performs **two separate heap allocations** (one for the object, one for the control block).
  - Using `std::make_shared<T>()` merges them into **a single contiguous heap allocation** (reducing malloc calls and improving CPU cache locality).

---

## 2. Thread Safety of `std::shared_ptr`

A common question in systems engineering interviews is the exact thread-safety guarantees of `std::shared_ptr`:

1. **Reference Count Operations are Thread-Safe**:
   - The reference counts inside the control block are modified using **atomic operations** (`std::atomic`).
   - Copying or destroying `shared_ptr` instances across threads simultaneously does not corrupt the count.
2. **The Managed Object is NOT Thread-Safe**:
   - If two threads access the same object's data members via different `shared_ptr` instances, you must synchronize access using a `std::mutex` or `std::lock_guard`.
3. **The `shared_ptr` Variable Itself is NOT Thread-Safe**:
   - If one thread modifies a `shared_ptr` variable (e.g. `ptr = otherPtr;`) while another thread reads it, this triggers **Undefined Behavior** (data race).
   - **C++20 Fix**: Use `std::atomic<std::shared_ptr<T>>` to perform atomic updates to the smart pointer variable itself.

```cpp
#include <memory>
#include <thread>
#include <mutex>

struct Data {
    int value = 0;
    std::mutex mtx;
};

void threadWorker(std::shared_ptr<Data> ptr) {
    // 1. Incrementing/decrementing strong count is thread-safe (done atomically)
    auto localCopy = ptr; 

    // 2. Modifying the managed data is NOT thread-safe: must use a lock!
    std::lock_guard<std::mutex> lock(localCopy->mtx);
    localCopy->value++;
}
```

---

## 3. `std::weak_ptr` (The Non-Owning Observer)

`std::weak_ptr` observes an object managed by `std::shared_ptr` **without incrementing its strong reference count**.
- Access is restricted: You cannot dereference `weak_ptr` directly. You must call **`lock()`** to obtain a temporary `shared_ptr`, checking if the object is still alive.

```cpp
#include <memory>
#include <iostream>

void demoWeak() {
    auto shared = std::make_shared<int>(100);
    std::weak_ptr<int> weak = shared;

    // Check if alive and retrieve shared access
    if (auto activeShared = weak.lock()) {
        std::cout << "Value: " << *activeShared << "\n";
    }

    shared.reset(); // Destroy object

    if (weak.expired()) {
        std::cout << "Object has been destroyed!\n";
    }
}
```

---

## 4. Breaking Cyclic References

If two objects hold `std::shared_ptr` references to each other, a cyclic loop is formed. The reference counts never drop to 0, causing a permanent memory leak.

```
       ┌──────────┐  shared_ptr   ┌──────────┐
       │ Parent   ├──────────────►│ Child    │
       │          │◄──────────────┤          │
       └──────────┘  shared_ptr   └──────────┘
             Both reference counts remain at 1 even when out of scope!
```

### The Fix:
Change the child-to-parent reference to a **`std::weak_ptr`**.

```cpp
#include <memory>

struct Child;

struct Parent {
    std::shared_ptr<Child> child;
};

struct Child {
    std::weak_ptr<Parent> parent; // Breaks the cycle!
};
```

---

*<- [[06_Mem_Unique_Ptr|Unique Pointers]] · [[06_Mem_Move_Semantics_and_Perfect_Forwarding|Move & Perfect Forwarding ->]]*
