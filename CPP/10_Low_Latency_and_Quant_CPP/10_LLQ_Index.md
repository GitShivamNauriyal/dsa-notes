---
tags: [cpp, quant, low-latency, index]
links: ["[[../00_Home]]", "[[../07_Concurrency/07_Conc_Index]]", "[[../08_Concurrency_Primitives_From_Scratch/08_CPS_Index]]", "[[../09_Performance_and_Optimization/09_Perf_Index]]"]
---

# Low-Latency & Quant C++ -- Index

*<- [[../09_Performance_and_Optimization/09_Perf_Index|Performance & Optimization]] · [[10_LLQ_Lock_Free_Queues_SPSC_MPSC\|01 · Lock-Free Queues (SPSC/MPSC) ->]]*

---

> [!IMPORTANT]
> **Prerequisites**: This chapter builds on advanced systems mechanics. Ensure you have fully digested:
> - `[[../07_Concurrency/07_Conc_Index]]` (Memory Models & Atomics)
> - `[[../08_Concurrency_Primitives_From_Scratch/08_CPS_Index]]` (Primitive Custom Building)
> - `[[../09_Performance_and_Optimization/09_Perf_Index]]` (Cache Locality & SSO/SBO)

---

| File | Topics |
|------|--------|
| [[10_LLQ_Lock_Free_Queues_SPSC_MPSC\|01 · Lock-Free Queues (SPSC/MPSC)]] | Single-Producer Single-Consumer (SPSC) and Multi-Producer Single-Consumer (MPSC) lock-free ring buffers |
| [[10_LLQ_Memory_Pools_and_Arena_Allocators\|02 · Memory Pools & Arena Allocators]] | Fixed-size block memory pools, stack-based arena allocators |
| [[10_LLQ_Avoiding_Allocations_in_Hot_Path\|03 · Avoiding Allocations in Hot Path]] | Pre-allocation strategies, dynamic heap allocation audits, low-latency code rules |
| [[10_LLQ_Fixed_Point_vs_Floating_Point\|04 · Fixed-Point vs Floating-Point]] | IEEE-754 precision issues, deterministic financial math fixed-point conversions |
| [[10_LLQ_NUMA_and_Kernel_Bypass_Concepts\|05 · NUMA & Kernel Bypass Concepts]] | Non-Uniform Memory Access (NUMA), page faults, thread affinity, DPDK/Solarflare onload kernel bypass |
| [[10_LLQ_Real_Time_Constraints_and_Jitter\|06 · Real-Time Constraints & Jitter]] | Operating system jitter causes, page fault traps, isolcpus, priority inheritance |
| [[10_LLQ_Problems\|07 · Concept Checks & Low-Latency Problems]] | Systems design and quant programming exercises |
| [[10_LLQ_Solutions\|08 · Worked Solutions]] | Complete solutions to the low-latency code challenges |
| [[10_LLQ_Tricky\|09 · Tricky Low-Latency Gotchas]] | Lock-free vs Mutex contention realities, warm-up HFT benchmarking skew, float non-determinism |

---

*<- [[../09_Performance_and_Optimization/09_Perf_Index|Performance & Optimization]] · [[10_LLQ_Lock_Free_Queues_SPSC_MPSC\|01 · Lock-Free Queues (SPSC/MPSC) ->]]*
