---
tags: [cpp, concurrency, index]
links: ["[[../00_Home]]", "[[../06_Memory_and_Smart_Pointers/06_Mem_Index]]"]
---

# Concurrency in C++ -- Index

*<- [[../06_Memory_and_Smart_Pointers/06_Mem_Index|Memory & Smart Pointers]] · [[07_Conc_Threading_Basics\|01 · Threading Basics ->]]*

---

| File | Topics |
|------|--------|
| [[07_Conc_Threading_Basics\|01 · Threading Basics]] | `std::thread`, `std::jthread` (C++20), cooperative cancellation, thread lifetimes, passing arguments |
| [[07_Conc_Synchronization_Primitives\|02 · Synchronization Primitives]] | Mutexes, `std::lock_guard`, `std::unique_lock`, `std::scoped_lock` (C++17), condition variables, deadlock avoidance |
| [[07_Conc_Atomics_and_Memory_Model\|03 · Atomics & Memory Model]] | `std::atomic`, compare-and-swap (CAS), cache lines, false sharing, memory orders (`relaxed`, `acquire`, `release`, `seq_cst`) |
| [[07_Conc_Semaphores_Latches_Barriers\|04 · Semaphores, Latches, & Barriers]] | C++20 concurrency primitives: `std::counting_semaphore`, `std::binary_semaphore`, `std::latch`, `std::barrier` |
| [[07_Conc_Lock_Free_Basics\|05 · Lock-Free Basics]] | Lock-free structures, ABA problem, lock-free stack design |
| [[07_Conc_Async_Future_Promise\|06 · Async, Future, & Promise]] | `std::async` policies, `std::future`, `std::promise`, `std::packaged_task` |
| [[07_Conc_Problems\|07 · Concept Checks & Concurrency Problems]] | Classic multithreading coordinates, queues, and order-of-execution exercises |
| [[07_Conc_Solutions\|08 · Worked Solutions]] | Complete solutions to the concurrency exercises |
| [[07_Conc_Tricky\|09 · Tricky Concurrency Gotchas]] | Destructor thread exceptions, atomic traps, volatile misconceptions |

---

*<- [[../06_Memory_and_Smart_Pointers/06_Mem_Index|Memory & Smart Pointers]] · [[07_Conc_Threading_Basics\|01 · Threading Basics ->]]*
