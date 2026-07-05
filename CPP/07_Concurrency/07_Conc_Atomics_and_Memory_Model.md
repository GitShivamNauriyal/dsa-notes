---
tags: [cpp, concurrency, atomics, cas, false-sharing, memory-orders]
links: ["[[07_Conc_Index]]", "[[07_Conc_Synchronization_Primitives]]", "[[07_Conc_Semaphores_Latches_Barriers]]"]
---

# Concurrency in C++ -- Atomics & Memory Model

*<- [[07_Conc_Synchronization_Primitives|Synchronization Primitives]] · [[07_Conc_Semaphores_Latches_Barriers|Semaphores, Latches & Barriers ->]]*

---

## 1. `std::atomic` and Compare-And-Swap (CAS)

`std::atomic<T>` provides atomic operations that are completed without the overhead of standard OS mutexes. The core synchronization primitive of lock-free programming is **Compare-And-Swap (CAS)**.

### `compare_exchange_weak` vs `compare_exchange_strong`
- **`compare_exchange_strong`**: Returns `true` if the atomic variable matches the expected value, replacing it with the desired value. Returns `false` otherwise.
- **`compare_exchange_weak`**: Can fail **spuriously** (returning `false` even if the value matches the expected value), but compiles down to much faster loop instructions on Load-Linked/Store-Conditional (LL/SC) architectures like ARM. It should be used inside loops where failure is handled by retrying.

```cpp
#include <atomic>

struct Node {
    int data;
    Node* next;
    Node(int val) : data(val), next(nullptr) {}
};

class LockFreeStack {
    std::atomic<Node*> head{nullptr};
public:
    void push(int val) {
        Node* newNode = new Node(val);
        
        // Load the current head
        newNode->next = head.load();
        
        // Loop CAS: if head matches newNode->next, set head to newNode.
        // If it fails (head changed), head.load() is updated into newNode->next and we retry.
        // We use 'weak' because it is faster inside this retry loop!
        while (!head.compare_exchange_weak(newNode->next, newNode)) {
            // newNode->next is automatically updated with the new head on failure!
        }
    }
};
```

---

## 2. Memory Ordering

Compilers and CPUs aggressively reorder read/write instructions to optimize instruction pipelines. Atomic operations can be constrained using `std::memory_order` flags to control reordering visibility.

### Memory Order Classes:
1. **`memory_order_relaxed`**:
   - Only the operation itself is atomic. No synchronization or ordering constraints are imposed on surrounding reads/writes.
2. **`memory_order_acquire` (Read)**:
   - No subsequent reads or writes can be reordered *before* this acquire barrier. Ensures you see all writes that occurred before the corresponding `release` write.
3. **`memory_order_release` (Write)**:
   - No previous reads or writes can be reordered *after* this release barrier. Synchronizes with an `acquire` read.
4. **`memory_order_seq_cst` (Sequentially Consistent)**:
   - The default. Implements a global total order of all atomic operations across all threads. High performance penalty.

### Acquire-Release Synchronization Example
```cpp
#include <atomic>
#include <thread>
#include <cassert>
#include <string>

std::atomic<bool> ready{false};
std::string data;

void producer() {
    data = "Payload";
    // Release write: ensures 'data' write is visible before ready is set to true
    ready.store(true, std::memory_order_release); 
}

void consumer() {
    // Acquire read: blocks subsequent reads from executing before ready becomes true
    while (!ready.load(std::memory_order_acquire)) {
        // Spin
    }
    // Guaranteed to see the string write!
    assert(data == "Payload"); 
}
```

---

## 3. False Sharing & Alignment

### The Problem: Cache Line Interference
CPUs do not load individual bytes from RAM; they load memory in chunks called **cache lines** (typically 64 bytes).
- If Thread 1 on Core A modifies `var1`, and Thread 2 on Core B modifies `var2`, and both variables sit on the **same cache line**, Core A's write invalidates the cache line on Core B.
- This forces the cores to repeatedly ping-pong the cache line back and forth, degrading performance. This is called **False Sharing**.

### The C++17 Solution: `std::hardware_destructive_interference_size`
Mark independent variables using `alignas` to ensure they are padded onto separate cache lines.

```cpp
#include <new>
#include <thread>

struct ThreadCounters {
    // Force variables onto separate cache lines to prevent false sharing!
    alignas(std::hardware_destructive_interference_size) int count1 = 0;
    alignas(std::hardware_destructive_interference_size) int count2 = 0;
};
```

---

*<- [[07_Conc_Semaphores_Latches_Barriers|Semaphores, Latches & Barriers]] · [[07_Conc_Semaphores_Latches_Barriers|Semaphores, Latches & Barriers ->]]*
