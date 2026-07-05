---
tags: [cpp, concurrency, lock-free, aba-problem, hazard-pointers, memory-reclamation]
links: ["[[07_Conc_Index]]", "[[07_Conc_Semaphores_Latches_Barriers]]", "[[07_Conc_Async_Future_Promise]]"]
---

# Concurrency in C++ -- Lock-Free Basics

*<- [[07_Conc_Semaphores_Latches_Barriers|Semaphores, Latches & Barriers]] · [[07_Conc_Async_Future_Promise|Async, Future & Promise ->]]*

---

## 1. What is Lock-Free Programming?

A concurrent data structure is **lock-free** if it guarantees that **at least one thread makes progress** in a finite number of steps.
- Unlike lock-based code (which suspends threads using mutexes, risking priority inversions and deadlocks), lock-free code relies on atomic hardware instructions (primarily Compare-And-Swap loops).

---

## 2. The ABA Problem

The **ABA Problem** is a structural hazard in lock-free algorithms that reuse memory addresses.

### The Race Condition Sequence (ASCII Timeline)

```
  Thread 1 (Pops)                                Thread 2 (Writer)
 ─────────────────                              ───────────────────
  1. Reads Head = A, Next = B
                                                2. Pops A
                                                3. Pops B (Deletes B)
                                                4. Allocates node (Reuses address A!)
                                                5. Pushes A (with new next = C)
  6. Resumes CAS(A, B)
     - Sees Head is still address A!
     - CAS succeeds!
     - Head is set to B (Dangling Pointer!)
```

### Explanation of the Corrupted Stack:
- Thread 1 wants to pop the head node `A`. It reads the current head `A` and the next node pointer `B` (`A->next = B`).
- Thread 1 is suspended right before executing its CAS step.
- Thread 2 runs, pops `A` and `B` from the stack, and deletes `B`.
- Thread 2 then allocates a new node. Because the allocator reuses freed addresses, the new node gets the exact same memory address `A`. Thread 2 pushes `A` back onto the stack.
- Thread 1 resumes and performs `head.compare_exchange(A, B)`.
- Because the address matches `A`, the CAS succeeds!
- Thread 1 sets `head` to `B`. But `B` was already deleted by Thread 2! The stack is now corrupted with a dangling head pointer pointing to deallocated memory.

---

## 3. Custom Lock-Free Stack Pop Implementation

Here is the implementation of a lock-free stack `pop` demonstrating the exact place where the ABA race condition occurs.

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
        newNode->next = head.load();
        while (!head.compare_exchange_weak(newNode->next, newNode)) {}
    }

    bool pop(int& result) {
        Node* oldHead = head.load();
        
        while (oldHead != nullptr) {
            // ---> ABA PITFALL LOCATION <---
            // If Thread 1 is suspended right here, and another thread
            // deletes oldHead, re-allocates it, and deletes oldHead->next,
            // 'next' below becomes a dangling pointer!
            Node* nextNode = oldHead->next; 
            
            if (head.compare_exchange_weak(oldHead, nextNode)) {
                result = oldHead->data;
                delete oldHead; // Safe to delete only if ABA is resolved!
                return true;
            }
            // oldHead is updated with the new head on failure, loop retries
        }
        return false; // Stack was empty
    }
};
```

---

## 4. Mitigations for the ABA Problem

1. **Tagged Pointers / Versioning**:
   - Associate a version counter with the pointer. Since the version increments on every write, `A (version 1)` does not equal `A (version 2)`.
   - In 64-bit systems, you can pack a 16-bit version tag into the unused upper bits of the pointer.
2. **Hazard Pointers**:
   - A thread registers the pointer it is currently reading in a thread-local "hazard pointer" slot.
   - The memory reclaimer checks this list before deleting any node. If a node is flagged as a hazard pointer, its deletion is deferred until all threads release their flags.
3. **Epoch-Based Reclamation (EBR)**:
   - Tracks global and thread-local epochs. Memory is only reclaimed when all active threads have transitioned out of the epoch in which the memory was marked for deletion.

---

*<- [[07_Conc_Semaphores_Latches_Barriers|Semaphores, Latches & Barriers]] · [[07_Conc_Async_Future_Promise|Async, Future & Promise ->]]*
