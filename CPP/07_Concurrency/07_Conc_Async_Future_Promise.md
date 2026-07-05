---
tags: [cpp, concurrency, async, future, promise, packaged-task, launch-policies]
links: ["[[07_Conc_Index]]", "[[07_Conc_Lock_Free_Basics]]", "[[07_Conc_Problems]]"]
---

# Concurrency in C++ -- Async, Future, & Promise

*<- [[07_Conc_Lock_Free_Basics|Lock-Free Basics]] · [[07_Conc_Problems|Problems ->]]*

---

C++11 introduced tasks-based concurrency using futures, promises, and packaged tasks to enable asynchronous calculations without managing raw threads directly.

---

## 1. `std::promise` and `std::future`

`std::promise` and `std::future` form a one-channel communication pipeline between threads.
- **`std::promise` (Write Channel)**: The producer thread sets the value or exception using `.set_value()` or `.set_exception()`.
- **`std::future` (Read Channel)**: The consumer thread calls `.get()` to retrieve the value. Calling `.get()` blocks the thread until the promise is fulfilled.
- **Rule**: You can only call `.get()` **once** on a standard `std::future`. If you need multiple threads to read the same value, copy it into a **`std::shared_future`**.

```cpp
#include <future>
#include <thread>
#include <iostream>

void calculateSquare(std::promise<int> valPromise, int x) {
    int res = x * x;
    std::this_thread::sleep_for(std::chrono::milliseconds(500));
    valPromise.set_value(res); // Fulfill the promise
}

void demoPromiseFuture() {
    std::promise<int> squarePromise;
    std::future<int> squareFuture = squarePromise.get_future();

    // Pass the promise by value (or move it)
    std::thread t(calculateSquare, std::move(squarePromise), 12);
    
    // Block until thread fulfills the promise
    int result = squareFuture.get(); 
    std::cout << "Result: " << result << "\n"; // Outputs 144
    
    t.join();
}
```

---

## 2. `std::async` Launch Policies

`std::async` runs a function asynchronously and returns a `std::future` to retrieve the output.

### The Two Launch Policies:
1. **`std::launch::async`**: Guarantees the task is executed on a **new, separate thread**.
2. **`std::launch::deferred`**: Task is evaluated **lazily** on the calling thread when `.get()` or `.wait()` is invoked on the future.
3. **Default Policy (`std::launch::async | std::launch::deferred`)**: The compiler decides at runtime depending on resource availability. This can lead to non-deterministic execution paths.

---

## 3. The `std::async` Temporary Future Destructor Trap

> [!WARNING]
> The `std::future` returned by `std::async` **blocks in its destructor** until the asynchronous task completes.
> If you call `std::async` without capturing the return value, the temporary future object is destroyed at the end of the statement, **blocking execution** and defeating the purpose of asynchronous concurrency!

```cpp
#include <future>
#include <thread>
#include <iostream>

void slowTask() {
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::cout << "Task complete\n";
}

void buggyAsync() {
    // Return future is NOT saved!
    // Temporary future destructor runs at the semicolon, blocking for 2 seconds!
    // This runs completely SYNCHRONOUSLY!
    std::async(std::launch::async, slowTask); 
    
    std::cout << "This print is blocked and executes AFTER the slowTask!\n";
}

void correctAsync() {
    // Future is captured: execution runs in the background.
    auto fut = std::async(std::launch::async, slowTask); 
    
    std::cout << "This print executes IMMEDIATELY while slowTask runs in background!\n";
    // fut destructor blocks at the end of the function block
}
```

---

## 4. `std::packaged_task`

`std::packaged_task<Signature>` wraps any callable target (functions, lambdas) so that it can be executed asynchronously. It automatically packages its return value or exceptions into a `std::future`.

```cpp
#include <future>
#include <thread>
#include <iostream>

int sum(int a, int b) { return a + b; }

void demoPackagedTask() {
    std::packaged_task<int(int, int)> task(sum);
    std::future<int> fut = task.get_future();

    // Run task on a separate thread
    std::thread t(std::move(task), 10, 20);

    std::cout << "Sum: " << fut.get() << "\n"; // Outputs 30
    t.join();
}
```

---

*<- [[07_Conc_Lock_Free_Basics|Lock-Free Basics]] · [[07_Conc_Problems|Problems ->]]*
