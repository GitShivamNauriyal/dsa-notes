---
tags: [cpp, low-latency, real-time, jitter, priority-inversion, scheduling]
links: ["[[10_LLQ_Index]]", "[[10_LLQ_NUMA_and_Kernel_Bypass_Concepts]]", "[[10_LLQ_Problems]]"]
---

# Low-Latency & Quant C++ -- Real-Time Constraints & Jitter

*<- [[10_LLQ_NUMA_and_Kernel_Bypass_Concepts|NUMA & Kernel Bypass Concepts]] · [[10_LLQ_Problems|Problems ->]]*

---

Low-latency execution requires eliminating **Jitter**—the variance in execution times. Even if your average latency is $2\mu\text{s}$, a single spike of $1\text{ms}$ (the **Tail Latency**) can result in missed market opportunities or canceled orders.

---

## 1. Operating System Jitter & Mitigations

### A. CPU Frequency Scaling (Power Saving Governors)
Modern CPUs downclock or sleep during quiet market periods to save power. When a market burst arrives, the CPU takes $\sim 20\mu\text{s}$ to spin back up to max frequency, creating a massive latency spike.
- **Mitigation**: Configure the OS scaling governor to **`performance`** (keeps CPU cores locked at max frequency constantly).

### B. Core Isolation (`isolcpus`)
The OS scheduler routinely interrupts threads to run housekeeping tasks, check timers, or schedule other processes.
- **Mitigation**: Boot the Linux kernel with the command line parameter **`isolcpus=2,3`** to completely remove cores 2 and 3 from the OS scheduling pool. The OS will never schedule standard processes on these cores. You can then pin your hot-path threads to them using thread affinity.

---

## 2. Real-Time Scheduling & Priority Inversion

In low-latency systems, critical threads are scheduled using real-time policies like **`SCHED_FIFO`** to preempt ordinary processes instantly. However, this introduces the threat of **Priority Inversion**.

### Priority Inversion Scenario:
1. Low-priority Thread L acquires Mutex A.
2. High-priority Thread H starts, preempts L, and tries to acquire Mutex A. Since A is locked, H blocks.
3. Medium-priority Thread M starts. Since M has higher priority than L (and H is suspended), M preempts L and runs.
4. **The Deadlock**: Thread L cannot run to release Mutex A because M is running. Thread H remains blocked by Mutex A. Thus, medium-priority Thread M effectively blocks high-priority Thread H!

```
  H (High)   ───► Blocks on Mutex A ───────────────────────────► (Starved!)
  M (Medium)                      ───► Preempts L & Runs ──────►
  L (Low)    ───► Locks Mutex A ──x (Preempted by M)
```

### The Fix: Priority Inheritance Protocol
Configure mutexes with the **Priority Inheritance** protocol (`PTHREAD_PRIO_INHERIT`). 
- When Thread H blocks on Mutex A, the OS temporarily elevates Thread L's priority to match H's priority. This prevents Thread M from preempting L, allowing L to run, release Mutex A, and restore its original priority.

```cpp
#if defined(__linux__)
#include <pthread.h>

pthread_mutex_t createPriorityInheritanceMutex() {
    pthread_mutex_t mutex;
    pthread_mutexattr_t attr;
    
    pthread_mutexattr_init(&attr);
    // Enable Priority Inheritance protocol
    pthread_mutexattr_setprotocol(&attr, PTHREAD_PRIO_INHERIT);
    
    pthread_mutex_init(&mutex, &attr);
    pthread_mutexattr_destroy(&attr);
    
    return mutex;
}
#endif
```

---

*<- [[10_LLQ_NUMA_and_Kernel_Bypass_Concepts|NUMA & Kernel Bypass Concepts]] · [[10_LLQ_Problems|Problems ->]]*
