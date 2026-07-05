---
tags: [cpp, low-latency, problems, exercises]
links: ["[[10_LLQ_Index]]", "[[10_LLQ_Real_Time_Constraints_and_Jitter]]", "[[10_LLQ_Solutions]]"]
---

# Low-Latency & Quant C++ -- Problems

*<- [[10_LLQ_Real_Time_Constraints_and_Jitter|Real-Time Constraints & Jitter]] · [[10_LLQ_Solutions|Solutions ->]]*

---

Implement low-latency primitives and structures matching the following engineering specifications.

---

### Problem 1: Lock-Free SPSC Queue Ring Buffer
Write a class `LockFreeSPSC` of generic type `T` and capacity `Cap` (must be a power of 2) that:
- Implements a single-producer single-consumer queue using a ring buffer.
- Employs lock-free atomic variables for read/write indexing with acquire-release memory ordering (no mutexes, no CAS).
- Isolates indices onto separate cache lines to prevent false sharing.

---

### Problem 2: Fixed-Size Block Memory Pool
Write a class `FixedBlockPool` that:
- Pre-allocates a pool of $N$ blocks of type `T` on construction.
- Exposes `T* allocate(Args&&...)` to construct objects in-place in $O(1)$ time.
- Exposes `void deallocate(T* ptr)` to destruct objects and reclaim blocks in $O(1)$ time.
- Prevents heap fragmentation entirely.

---

### Problem 3: Hot-Path Memory Allocation Auditor
Create a utility scope class `HotPathAuditor` that:
- Uses a thread-local flag to monitor memory operations.
- Overloads global `operator new` and `operator delete` to raise an error or call `std::abort()` if any dynamic allocation/deallocation occurs while the auditor is active.

---

### Problem 4: Fixed-Point Currency Class
Write a class `CurrencyDec4` that:
- Represents dollar amounts scaled to 4 decimal places of precision using `std::int64_t` under the hood.
- Implements deterministic addition, subtraction, and multiplication operators.
- Prevents accumulation of floating-point rounding errors.

---

*<- [[10_LLQ_Real_Time_Constraints_and_Jitter|Real-Time Constraints & Jitter]] · [[10_LLQ_Solutions|Solutions ->]]*
