---
tags: [cpp, low-latency, numa, thread-affinity, kernel-bypass, dpdk, page-faults]
links: ["[[10_LLQ_Index]]", "[[10_LLQ_Fixed_Point_vs_Floating_Point]]", "[[10_LLQ_Real_Time_Constraints_and_Jitter]]"]
---

# Low-Latency & Quant C++ -- NUMA & Kernel Bypass Concepts

*<- [[10_LLQ_Fixed_Point_vs_Floating_Point|Fixed-Point vs Floating-Point]] · [[10_LLQ_Real_Time_Constraints_and_Jitter|Real-Time Constraints & Jitter ->]]*

---

Low-latency systems must be highly optimized to interact efficiently with operating system internals and network hardware.

---

## 1. NUMA (Non-Uniform Memory Access)

In multi-socket server hardware, memory is partitioned. A CPU socket has fast access to its directly connected **local memory bank**, but slower access (adding $\sim 50\text{ns}$ latency) when fetching from memory banks connected to **remote sockets** over interconnect buses (QPI/UPI).

### Thread Affinity (Thread Pinning)
To maximize L1/L2 cache efficiency and avoid remote NUMA accesses, threads are pinned to specific CPU cores. This prevents the OS scheduler from migrating threads across different cores or sockets.

```cpp
#if defined(__linux__)
#include <pthread.h>
#include <sched.h>

void pinThreadToCore(int coreId) {
    cpu_set_t cpuset;
    CPU_ZERO(&cpuset);
    CPU_SET(coreId, &cpuset);
    
    pthread_t currentThread = pthread_self();
    // Pin the thread to the specified core
    pthread_setaffinity_np(currentThread, sizeof(cpu_set_t), &cpuset);
}
#endif
```

---

## 2. Preventing Lazy Page Faults (Memory Warm-up)

When you allocate memory using `new` or `malloc`, the OS does not immediately map physical RAM pages to your pointer. Instead, it lazily registers a virtual memory address.
- **The Trap**: The first time you write to that memory inside your hot path, the CPU triggers a **Page Fault** interrupt. The OS halts your program, maps a physical page, and resumes execution, adding a massive **$\sim 10\mu\text{s}$ latency spike**.
- **The Fix**: **Warm up** your memory buffers at startup during the initialization phase by writing to every page.

```cpp
#include <vector>
#include <cstring>

void allocateAndWarmUp(std::vector<char>& buffer, std::size_t size) {
    buffer.resize(size);
    // Write 0 to every page (typically 4096 bytes) to trigger all page faults at startup!
    std::memset(buffer.data(), 0, size); 
}
```

---

## 3. Kernel Bypass Concepts

In a standard OS network stack, a network packet from a NIC (Network Interface Card) undergoes a slow process:
1. NIC raises a hardware interrupt.
2. OS kernel copies packet into kernel buffer (socket buffer).
3. Context switch occurs to copy data from kernel space to user space.
- This pipeline introduces up to **$5\text{--}10\mu\text{s}$ of latency**.

```
  Standard Stack (Slow):
  [ NIC ] ──► [ OS Kernel Socket Buffer ] (interrupt) ──► [ User Application ]
  
  Kernel Bypass (Fast):
  [ NIC ] ──────────────────────────────────────────────► [ User Application ] (polling)
```

### The Kernel Bypass Solution
Kernel bypass libraries (like **DPDK** or **Solarflare EF_VI / Onload**) map the NIC's ring buffers directly into the user application's memory space.
- The application **polls** the NIC registers in a continuous spin loop, eliminating interrupts, kernel copies, and context switches.
- **Result**: Packet processing latency drops from microseconds to **nanoseconds**.

---

*<- [[10_LLQ_Fixed_Point_vs_Floating_Point|Fixed-Point vs Floating-Point]] · [[10_LLQ_Real_Time_Constraints_and_Jitter|Real-Time Constraints & Jitter ->]]*
