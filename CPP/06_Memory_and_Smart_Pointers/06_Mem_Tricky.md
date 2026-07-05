---
tags: [cpp, memory, tricky, make-shared, enable-shared-from-this, aliasing-constructor]
links: ["[[06_Mem_Index]]", "[[06_Mem_Solutions]]", "[[../07_Concurrency/07_Conc_Index]]"]
---

# Memory & Smart Pointers -- Tricky & Higher-Order

*<- [[06_Mem_Solutions|Solutions]] · [[../07_Concurrency/07_Conc_Index|Concurrency ->]]*

---

## 1. `std::enable_shared_from_this` & Constructor Pitfall

**The Problem**: A class member function needs to pass a `std::shared_ptr` to `this` to another function.
- If you write `std::shared_ptr<Widget>(this)`, the compiler creates a **new, independent control block** for the same object. When both control blocks think they are the sole owner, they will both call `delete this`, causing a double-free crash!

### The Fix:
Inherit from `std::enable_shared_from_this<T>` and call `shared_from_this()`, which queries the existing control block.

```cpp
#include <memory>
#include <iostream>

class Widget : public std::enable_shared_from_this<Widget> {
public:
    void registerCallback() {
        // Safe: shares the existing control block
        std::shared_ptr<Widget> safeThis = shared_from_this();
    }
};
```

### The Constructor Gotcha (Crash Trap)
> [!CAUTION]
> You **cannot** call `shared_from_this()` inside the constructor of the class! 
> During constructor execution, the object is still being created, and no `std::shared_ptr` owns it yet. Calling it throws a `std::bad_weak_ptr` exception.

```cpp
class BadWidget : public std::enable_shared_from_this<BadWidget> {
public:
    BadWidget() {
        // CRASH: Throws std::bad_weak_ptr! No shared_ptr owns the object yet!
        auto ptr = shared_from_this(); 
    }
};
```

---

## 2. Why `make_shared` is Usually Preferred, But Not Always

`std::make_shared` merges the allocation of the managed object and its control block into a **single contiguous block** on the heap (1 malloc instead of 2).

### The Contiguous Memory Trap (Lingering Memory Leak)
Because the object and the control block are in the same block, they cannot be freed independently:
- The managed object destructor is called as soon as the strong count drops to 0.
- However, the **heap memory** for the object cannot be released until the control block is deleted.
- The control block is only deleted when both the strong count AND the **weak count drop to 0**.
- **The Trap**: If you have a massive object (e.g. 10MB vector) and a long-lived `std::weak_ptr` observes it, the entire 10MB heap allocation remains locked in memory even after the object's lifetime has ended!

```
 make_shared allocation block:
 ┌───────────────────────────┬──────────────────────────┐
 │ Managed Object (10MB)     │ Control Block (24 bytes) │
 └───────────────────────────┴──────────────────────────┘
  Strong count = 0 (destructor run), but Weak count = 1.
  Linker cannot call free() on this block because Control Block is still needed!
```

- **Solution**: If you have huge objects and long-lived `weak_ptr` observers, allocate using `std::shared_ptr<T>(new T)` instead. This allocates them separately, allowing the 10MB object memory to be freed immediately when the strong count hits 0.

---

## 3. The Aliasing Constructor of `std::shared_ptr`

**Concept**: A `shared_ptr` constructor that shares ownership of object A (incrementing its count), but points to a member variable B inside A.
- Syntax: `std::shared_ptr<B>(std::shared_ptr<A> parent, B* member)`
- This is useful for passing around sub-components of a large structure safely, ensuring the parent structure remains alive as long as any sub-component is being used.

```cpp
#include <memory>

struct Engine {};
struct Car { Engine eng; };

std::shared_ptr<Engine> getEngineFromCar(std::shared_ptr<Car> car) {
    // Aliasing constructor: shares ownership of 'car', but points to '&car->eng'
    return std::shared_ptr<Engine>(car, &car->eng);
}
```

---

## Recap of Tricky Memory Gotchas

| Gotcha | Theme |
|---|---|
| **`enable_shared_from_this`** | Calling it inside constructors throws `std::bad_weak_ptr` because no `shared_ptr` ownership exists yet. |
| **`make_shared` Lingering Memory** | Contiguous allocation locks the object's heap memory block as long as any `weak_ptr` remains alive. |
| **Aliasing Constructor** | Shares the control block of a parent object while pointing directly to an internal sub-component member. |

---

*<- [[06_Mem_Solutions|Solutions]] · [[../07_Concurrency/07_Conc_Index|Concurrency ->]]*
