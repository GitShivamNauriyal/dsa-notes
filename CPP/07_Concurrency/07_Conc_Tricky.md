---
tags: [cpp, concurrency, tricky, volatile, atomic-double-check, destructor-exceptions]
links: ["[[07_Conc_Index]]", "[[07_Conc_Solutions]]", "[[../08_Concurrency_Primitives_From_Scratch/08_Scratch_Index]]"]
---

# Concurrency in C++ -- Tricky & Higher-Order

*<- [[07_Conc_Solutions|Solutions]] · [[../08_Concurrency_Primitives_From_Scratch/08_Scratch_Index|Concurrency Primitives From Scratch ->]]*

---

## 1. The `volatile` Misconception in C++

Many programmers coming from Java or C# assume that the **`volatile`** keyword makes a variable safe for multithreaded synchronization.

> [!CAUTION]
> **In C++, `volatile` does NOT provide atomic operations and does NOT introduce memory barriers!**
> - It only tells the compiler: "This variable can change outside the program's control. Do not optimize out reads or writes to it."
> - It is intended exclusively for **memory-mapped I/O (MMIO)** or interacting with hardware devices.
> - Accessing a non-atomic `volatile` variable across multiple threads triggers a **data race** (Undefined Behavior).
> - For thread safety and atomic reads/writes, you **must** use **`std::atomic`**.

---

## 2. The Spurious Wakeup Condition Variable Trap

**The Trap**: Writing a condition variable wait statement without a check condition predicate.

```cpp
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void buggyWait() {
    std::unique_lock<std::mutex> lock(mtx);
    // Bug: Wakes up spuriously without ready being true!
    // The thread continues executing with invalid state.
    cv.wait(lock); 
}

void correctWait() {
    std::unique_lock<std::mutex> lock(mtx);
    // Correct: checks the predicate. If woken spuriously, it re-suspends
    cv.wait(lock, []() { return ready; }); 
}
```

---

## 3. Double-Checked Locking Optimization Trap

**The Trap**: Implementing a Singleton class using a raw pointer checked twice, but using relaxed atomic checks.

```cpp
#include <atomic>
#include <mutex>

class Singleton {
    static std::atomic<Singleton*> instance;
    static std::mutex mtx;
public:
    static Singleton* getInstance() {
        // First check (Relaxed check - BUG!)
        Singleton* tmp = instance.load(std::memory_order_relaxed);
        if (tmp == nullptr) {
            std::lock_guard<std::mutex> lock(mtx);
            tmp = instance.load(std::memory_order_relaxed);
            if (tmp == nullptr) {
                tmp = new Singleton();
                // Release write: makes constructor changes visible
                instance.store(tmp, std::memory_order_release);
            }
        }
        return tmp;
    }
};
```
- **Why it fails**: Because the first check uses `memory_order_relaxed`, it can be reordered *before* the constructor writes are completed. Another thread can read a non-null pointer but access half-constructed memory!
- **The Fix**: The first check **must** use `std::memory_order_acquire`.

---

## 4. `std::thread` Destructor Exceptions

If a `std::thread` variable is destroyed while it is still **joinable** (neither `join()` nor `detach()` was called), its destructor calls `std::terminate()`, crashing the program.

- **Reason**: The C++ standard designers chose to terminate the program rather than silently join or detach. Silently joining can cause hang lockups (destructor blocks forever), and silently detaching can cause dangling references (threads write to stack variables of the caller scope which already exited).

---

## Recap of Tricky Concurrency Gotchas

| Gotcha | Theme |
|---|---|
| **`volatile` Misconception** | `volatile` is for MMIO hardware access; it lacks atomic properties and memory barriers. Use `std::atomic`. |
| **Spurious Wakeup** | Condition variables can wake up without a signal; waiting must always be protected with a loop predicate check. |
| **Double-Checked Locking**| Requires `memory_order_acquire` on the initial read to ensure constructor allocations are fully synchronized. |
| **Destructor Crash** | Destroying a `std::thread` while it is joinable calls `std::terminate()` to prevent dangling references or silent blocks. |

---

*<- [[07_Conc_Solutions|Solutions]] · [[../08_Concurrency_Primitives_From_Scratch/08_Scratch_Index|Concurrency Primitives From Scratch ->]]*
