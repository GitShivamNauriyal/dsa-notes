---
tags: [cpp, concurrency, scratch, read-write-lock, writer-starvation, fairness]
links: ["[[08_CPS_Index]]", "[[08_CPS_Semaphore]]", "[[08_CPS_Bounded_Blocking_Queue]]"]
---

# Concurrency Primitives From Scratch -- Read-Write Locks

*<- [[08_CPS_Semaphore|Semaphores]] · [[08_CPS_Bounded_Blocking_Queue|Bounded Blocking Queues ->]]*

---

A **Read-Write (RW) Lock** optimizes access for structures where reads are frequent but writes are rare.

---

## 1. Reader-Preferred RW Lock

### The Starvation Hazard
In a reader-preferred lock, newly arriving readers can acquire the lock as long as a writer is not actively writing. If readers arrive in a continuous stream, the reader count never drops to 0. A waiting writer remains blocked indefinitely (**Writer Starvation**).

```cpp
#include <mutex>
#include <condition_variable>

class ReaderPreferredRWLock {
    std::mutex mtx;
    std::condition_variable cv;
    int activeReaders = 0;
    bool writerActive = false;

public:
    void lockRead() {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [this]() { return !writerActive; });
        activeReaders++;
    }

    void unlockRead() {
        std::unique_lock<std::mutex> lock(mtx);
        activeReaders--;
        if (activeReaders == 0) {
            cv.notify_all(); // Wake up waiting writers
        }
    }

    void lockWrite() {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [this]() { return activeReaders == 0 && !writerActive; });
        writerActive = true;
    }

    void unlockWrite() {
        std::unique_lock<std::mutex> lock(mtx);
        writerActive = false;
        cv.notify_all();
    }
};
```

---

## 2. Writer-Preferred RW Lock

To prevent writer starvation, if any writers are waiting, newly arriving readers are forced to wait. Arriving readers queue up behind the pending writer.

```cpp
#include <mutex>
#include <condition_variable>

class WriterPreferredRWLock {
    std::mutex mtx;
    std::condition_variable cv;
    int activeReaders = 0;
    int waitingWriters = 0;
    bool writerActive = false;

public:
    void lockRead() {
        std::unique_lock<std::mutex> lock(mtx);
        // Force readers to wait if a writer is active OR if any writers are waiting!
        cv.wait(lock, [this]() { return !writerActive && waitingWriters == 0; });
        activeReaders++;
    }

    void unlockRead() {
        std::unique_lock<std::mutex> lock(mtx);
        activeReaders--;
        if (activeReaders == 0) {
            cv.notify_all(); // Wake up waiting writers
        }
    }

    void lockWrite() {
        std::unique_lock<std::mutex> lock(mtx);
        waitingWriters++;
        cv.wait(lock, [this]() { return activeReaders == 0 && !writerActive; });
        waitingWriters--;
        writerActive = true;
    }

    void unlockWrite() {
        std::unique_lock<std::mutex> lock(mtx);
        writerActive = false;
        cv.notify_all(); // Wake up queued readers or writers
    }
};
```

---

## 3. Starvation-Free (Fair) RW Lock

To implement a completely fair RW lock, we can use a **ticket or phase system**. Arriving threads (readers or writers) obtain a ticket. Execution is coordinated in strict ticket order, ensuring no thread is starved.

```cpp
#include <mutex>
#include <condition_variable>

class StarvationFreeRWLock {
    std::mutex mtx;
    std::condition_variable cv;
    
    // Ticket Counters
    unsigned int nextTicket = 0;
    unsigned int activeTicket = 0;
    
    int activeReaders = 0;
    bool writerActive = false;

public:
    void lockRead() {
        std::unique_lock<std::mutex> lock(mtx);
        unsigned int myTicket = nextTicket++;
        
        // Wait until it is our ticket's turn, and no writer is active
        cv.wait(lock, [this, myTicket]() {
            return myTicket == activeTicket && !writerActive;
        });
        
        activeReaders++;
        activeTicket++; // Pass the turn to the next ticket in line
        cv.notify_all();
    }

    void unlockRead() {
        std::unique_lock<std::mutex> lock(mtx);
        activeReaders--;
        if (activeReaders == 0) {
            cv.notify_all(); // Wake up waiting writer
        }
    }

    void lockWrite() {
        std::unique_lock<std::mutex> lock(mtx);
        unsigned int myTicket = myTicket = nextTicket++;
        
        // Writer must wait until its ticket turn, all readers finish, and no active writer
        cv.wait(lock, [this, myTicket]() {
            return myTicket == activeTicket && activeReaders == 0 && !writerActive;
        });
        
        writerActive = true;
    }

    void unlockWrite() {
        std::unique_lock<std::mutex> lock(mtx);
        writerActive = false;
        activeTicket++; // Pass turn to the next ticket in line
        cv.notify_all();
    }
};
```

---

*<- [[08_CPS_Semaphore|Semaphores]] · [[08_CPS_Bounded_Blocking_Queue|Bounded Blocking Queues ->]]*
