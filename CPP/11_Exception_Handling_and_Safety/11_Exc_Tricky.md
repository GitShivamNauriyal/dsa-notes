---
tags: [cpp, exceptions, tricky, throwing-destructors, exceptions-across-threads, std-exception-ptr]
links: ["[[11_Exc_Index]]", "[[11_Exc_Solutions]]", "[[../00_Home]]"]
---

# Exception Handling & Safety -- Tricky & Higher-Order

*<- [[11_Exc_Solutions|Solutions]] · [[../00_Home|Home ->]]*

---

## 1. Throwing in Destructors during Unwinding

**The Gotcha**: If a destructor throws an exception while the runtime is *already* unwinding the stack to process another active exception, C++ calls **`std::terminate()`** immediately, crashing the program.

- **Rule**: Destructors in C++11 and later are **implicitly `noexcept`**. 
- If a destructor must perform operations that can throw (e.g. flushing a database buffer), it **must** catch and handle all exceptions internally.

```cpp
#include <iostream>
#include <stdexcept>

class BadDestructor {
public:
    ~BadDestructor() {
        // Throwing inside a destructor is highly dangerous!
        throw std::runtime_error("Error inside destructor"); 
    }
};

void demoCrash() {
    try {
        BadDestructor d;
        throw std::runtime_error("First error"); // Stack unwinding begins
    } catch (...) {
        // Never reached: during unwinding of 'First error', d's destructor throws
        // 'Error inside destructor'. The program terminates instantly!
    }
}
```

---

## 2. Exceptions Across Thread Boundaries

**The Gotcha**: Exceptions thrown in worker threads **cannot** be caught by the parent thread using standard try-catch blocks. If left uncaught in the worker thread, it crashes the entire application.

### The Solution: `std::exception_ptr`
You can capture exceptions in the worker thread using **`std::current_exception()`** and save them in a **`std::exception_ptr`**, then pass it to the main thread to be re-thrown using **`std::rethrow_exception()`**.

```cpp
#include <thread>
#include <exception>
#include <iostream>

// Shared exception pointer
std::exception_ptr workerException = nullptr;

void workerTask() {
    try {
        // Some logic that throws
        throw std::runtime_error("Worker thread failed!");
    } catch (...) {
        // Capture the active exception
        workerException = std::current_exception(); 
    }
}

void demoThreadExceptions() {
    std::thread t(workerTask);
    t.join();

    if (workerException) {
        try {
            // Rethrow the worker's exception in the main thread!
            std::rethrow_exception(workerException); 
        } catch (const std::exception& e) {
            std::cout << "Caught exception from worker thread: " << e.what() << "\n";
        }
    }
}
```

---

## Recap of Tricky Exception Gotchas

| Gotcha | Theme |
|---|---|
| **Throwing Destructors** | If a destructor throws during stack unwinding, the runtime invokes `std::terminate()` immediately. |
| **Thread Boundaries** | Uncaught exceptions in worker threads crash the application; pass them using `std::exception_ptr`. |
| **Vector Move Fallback** | Vectors fallback to copying elements during reallocation if move constructors are not marked `noexcept`. |

---

*<- [[11_Exc_Solutions|Solutions]] · [[../00_Home|Home ->]]*
