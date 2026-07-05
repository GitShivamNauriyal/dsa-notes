---
tags: [cpp, concurrency, thread, jthread, stop-token, lifetimes]
links: ["[[07_Conc_Index]]", "[[07_Conc_Synchronization_Primitives]]"]
---

# Concurrency in C++ -- Threading Basics

*<- [[07_Conc_Index|Index]] · [[07_Conc_Synchronization_Primitives|Synchronization Primitives ->]]*

---

## 1. `std::thread` and Lifetimes

`std::thread` (C++11) is the standard class representing a thread of execution.

> [!WARNING]
> **The Destructor Crash Rule**: A `std::thread` object must either be **joined** (waiting for thread execution to finish via `.join()`) or **detached** (allowing thread to run independently in the background via `.detach()`) before the `std::thread` variable goes out of scope. 
> If a `std::thread` object is destroyed while it is still **joinable** (meaning neither `join()` nor `detach()` was called), its destructor immediately calls **`std::terminate()`**, crashing the entire program!

```cpp
#include <thread>
#include <iostream>

void task() {
    std::cout << "Thread running...\n";
}

void buggyThread() {
    std::thread t(task);
    // Destructor of t runs at the end of the block.
    // Since we did not call t.join() or t.detach(), the program crashes!
}

void correctThread() {
    std::thread t(task);
    t.join(); // Blocks parent execution until t finishes. Safe!
}
```

---

## 2. Argument Passing & Reference Semantics

By default, the constructor of `std::thread` copies or moves all arguments into the thread's internal storage.
- To pass an argument by reference, you **must** wrap it in **`std::ref`** (from `<functional>`). Passing raw references directly results in compilation failures.

```cpp
#include <thread>
#include <functional>
#include <iostream>

void updateValue(int& x) {
    x += 10;
}

void demoRef() {
    int val = 5;
    // std::thread t(updateValue, val); // COMPILER ERROR: cannot pass by value to reference parameter!
    
    std::thread t(updateValue, std::ref(val)); // Correct: wrap in std::ref
    t.join();
    std::cout << val << "\n"; // Outputs 15
}
```

---

## 3. `std::jthread` (C++20 Cooperative Joining)

C++20 introduced **`std::jthread`** (joining thread) to solve the lifetime usability traps of `std::thread`.

### Automatic Join
In its destructor, `std::jthread` automatically:
1. Signals cooperative cancellation (requests a stop).
2. Calls `join()` on the thread, blocking until execution terminates.
- This prevents destructor-termination crashes completely.

```cpp
#include <thread>

void taskJ() {
    // ...
}

void safeJThread() {
    std::jthread jt(taskJ);
    // jt goes out of scope here.
    // Destructor runs: automatically calls jt.join() and blocks. Safe!
}
```

---

## 4. Cooperative Cancellation using `std::stop_token`

In multithreaded systems, killing a thread abruptly from the outside is dangerous because it can leave mutexes permanently locked and resources leaked. C++20 solves this with **cooperative cancellation**.

- A thread function can accept a **`std::stop_token`** as its first parameter.
- The calling code can request a stop using `jthread::request_stop()`.
- The thread checks `token.stop_requested()` periodically inside its execution loop to clean up and exit gracefully.

```cpp
#include <thread>
#include <chrono>
#include <iostream>

void cancellableWorker(std::stop_token token) {
    while (!token.stop_requested()) {
        std::cout << "Working...\n";
        std::this_thread::sleep_for(std::chrono::milliseconds(200));
    }
    std::cout << "Stopping worker gracefully...\n";
}

void demoCancellation() {
    std::jthread jt(cancellableWorker);
    
    std::this_thread::sleep_for(std::chrono::seconds(1));
    
    // Request the thread to stop
    jt.request_stop(); 
    // jt goes out of scope and joins automatically
}
```

---

*<- [[07_Conc_Index|Index]] · [[07_Conc_Synchronization_Primitives|Synchronization Primitives ->]]*
