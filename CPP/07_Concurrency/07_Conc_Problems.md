---
tags: [cpp, concurrency, problems, exercises]
links: ["[[07_Conc_Index]]", "[[07_Conc_Async_Future_Promise]]", "[[07_Conc_Solutions]]"]
---

# Concurrency in C++ -- Problems & Exercises

*<- [[07_Conc_Async_Future_Promise|Async, Future & Promise]] · [[07_Conc_Solutions|Solutions ->]]*

---

## Tier 1 -- Coordination Basics

### Problem 1.1: Print in Order
Suppose we have a class:
```cpp
class Foo {
public:
    void first() { print("first"); }
    void second() { print("second"); }
    void third() { print("third"); }
};
```
The same instance of `Foo` will be passed to three different threads. Thread A will call `first()`, Thread B will call `second()`, and Thread C will call `third()`.
Design a mechanism using synchronization primitives to ensure that `first()` is always executed before `second()`, and `second()` is always executed before `third()`.

---

### Problem 1.2: Print FooBar Alternately
You are given a class:
```cpp
class FooBar {
private:
    int n;
public:
    FooBar(int n) { this->n = n; }
    void foo() {
        for (int i = 0; i < n; i++) {
            print("foo");
        }
    }
    void bar() {
        for (int i = 0; i < n; i++) {
            print("bar");
        }
    }
};
```
Two threads are spawned to run `foo()` and `bar()` concurrently. Write a thread-safe implementation of the class to ensure that the string `"foobar"` is printed exactly $n$ times alternately (e.g. `foobarfoobar...`).

---

## Tier 2 -- Advanced State Coordination

### Problem 2.1: Print Zero Even Odd
You have a class:
```cpp
class ZeroEvenOdd {
private:
    int n;
public:
    ZeroEvenOdd(int n) { this->n = n; }
    void zero() { ... } // prints 0
    void even() { ... } // prints even numbers
    void odd()  { ... } // prints odd numbers
};
```
Three threads are spawned to run these methods concurrently. For a given $n$, coordinate the threads to output the sequence `0102030405...` up to $n$ numbers.
- E.g. for $n = 5$, the output must be `0102030405`.

---

### Problem 2.2: Fizz Buzz Multithreaded
Design a thread-safe class `FizzBuzz` that coordinates four separate threads:
1. Thread A calls `fizz()` to output `"fizz"` if the number is divisible by 3 (and not 5).
2. Thread B calls `buzz()` to output `"buzz"` if the number is divisible by 5 (and not 3).
3. Thread C calls `fizzbuzz()` to output `"fizzbuzz"` if the number is divisible by both 3 and 5.
4. Thread D calls `number()` to output the number if it is not divisible by 3 or 5.
Coordinate the threads to output the standard FizzBuzz sequence from 1 to $n$.

---

## Tier 3 -- Real-World Systems Coordination

### Problem 3.1: Building H2O
We want to simulate the formation of water molecules. There are two types of threads: `hydrogen` and `oxygen`. 
To form a single water molecule, we need exactly two hydrogen atoms and one oxygen atom.
Write a class `H2O` that coordinates these threads:
- If an oxygen thread arrives, it must wait until two hydrogen threads are ready.
- If a hydrogen thread arrives, it must wait until another hydrogen and an oxygen thread are ready.
- Ensure that the execution of these threads completes in groups of three representing `H2O` (e.g. `HOH`, `HHO`, `OHH` are valid sequences of outputs).

---

### Problem 3.2: Dining Philosophers (Deadlock Prevention)
Five philosophers sit around a circular table. Each philosopher has a plate of spaghetti and a fork to their left and right.
A philosopher needs two forks to eat. When finished, they put down both forks.
Design a class `DiningPhilosophers` that coordinates five threads (representing the philosophers) to pick up forks and eat without deadlocks or starvation.

---

### Problem 3.3: Bounded Blocking Queue
Implement a thread-safe `BoundedBlockingQueue` of integers:
- The queue has a maximum capacity limit.
- `enqueue(int element)`: Adds an element to the queue, blocking the calling thread if the queue is full.
- `dequeue()`: Removes and returns an element from the queue, blocking the calling thread if the queue is empty.
- Avoid busy-waiting (spinning) completely.

---

## Tier 4 -- Systems-Hard (Original Challenges)

### Problem 4.1: Custom Priority-Aware Task Scheduler
Design a thread-safe task scheduler `PriorityScheduler` that coordinates worker threads:
- Threads can submit tasks with an integer priority level ($0 = \text{highest priority}$).
- Worker threads request tasks. The scheduler must always hand out the **highest-priority task** first.
- If multiple tasks share the same priority, resolve them in First-In-First-Out (FIFO) order.
- Workers must block if no tasks are available.

---

### Problem 4.2: N-Thread Cyclic Execution Chain
Write a program that spawns $N$ threads, numbered $0$ to $N-1$.
Coordinate the threads using C++20 synchronization primitives so that they execute their operations in a strict, recurring cyclic order: Thread 0 runs, then Thread 1, ..., then Thread $N-1$, then Thread 0 again, for $M$ complete cycles.
- Do not use spin loops. The synchronization overhead must be $O(1)$ per thread switch.

---

*<- [[07_Conc_Async_Future_Promise|Async, Future & Promise]] · [[07_Conc_Solutions|Solutions ->]]*
