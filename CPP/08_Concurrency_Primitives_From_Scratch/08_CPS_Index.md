---
tags: [cpp, concurrency, scratch, index]
links: ["[[../00_Home]]", "[[../07_Concurrency/07_Conc_Index]]"]
---

# Concurrency Primitives From Scratch -- Index

*<- [[../07_Concurrency/07_Conc_Index|Concurrency]] · [[08_CPS_Spinlock\|01 · Spinlocks ->]]*

---

> [!IMPORTANT]
> **Prerequisites**: Before reading this chapter, ensure you have fully digested the memory model, atomics, and ordering rules in `[[../07_Concurrency/07_Conc_Index]]`.

---

| File | Topics |
|------|--------|
| [[08_CPS_Spinlock\|01 · Spinlocks]] | CAS Spinlock, Exponential Backoff, Ticket Spinlock |
| [[08_CPS_Mutex\|02 · Mutexes]] | Custom Mutex, Recursive Mutex, Shared (RW) Mutex |
| [[08_CPS_Semaphore\|03 · Semaphores]] | Binary Semaphore, Counting Semaphore (Mutex/Atomics versions) |
| [[08_CPS_Read_Write_Lock\|04 · Read-Write Locks]] | Reader-Preferred, Writer-Preferred, Starvation-Free RW Locks |
| [[08_CPS_Bounded_Blocking_Queue\|05 · Bounded Blocking Queues]] | Mutex/Condvar, Semaphore-based, Lock-Free Ring Buffer |
| [[08_CPS_Thread_Pool\|06 · Thread Pools]] | Standard Thread Pool, Priority-Aware Pool, Work-Stealing Pool |
| [[08_CPS_Barrier_and_Latch\|07 · Barriers & Latches]] | Latch from scratch, Barrier from scratch |
| [[08_CPS_Problems\|08 · Concept Checks & Scratch Problems]] | Building custom primitives and coordinating threads |
| [[08_CPS_Solutions\|09 · Worked Solutions]] | Complete solutions to the custom primitives exercises |
| [[08_CPS_Tricky\|10 · Tricky Scratch Gotchas]] | Spurious wakeups on raw variables, lock-release orders |

---

*<- [[../07_Concurrency/07_Conc_Index|Concurrency]] · [[08_CPS_Spinlock\|01 · Spinlocks ->]]*
