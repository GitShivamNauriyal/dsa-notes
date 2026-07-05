---
tags: [cpp, performance, tricky, false-sharing, vector-reallocations, benchmark-invalidation]
links: ["[[09_Perf_Index]]", "[[09_Perf_Solutions]]", "[[../00_Complexity_Cheatsheet]]", "[[../10_Low_Latency_and_Quant_CPP/10_LLQ_Index]]"]
---

# Performance & Optimization -- Tricky & Higher-Order

*<- [[09_Perf_Solutions|Solutions]] · [[../10_Low_Latency_and_Quant_CPP/10_LLQ_Index|Low-Latency & Quant C++ ->]]*

---

## 1. False Sharing on Atomic Counters

**The Gotcha**: Modifying two independent atomic variables in different threads can bottleneck performance if the variables reside on the same cache line.

```cpp
#include <atomic>
#include <thread>

struct SharedData {
    // Both variables sit in the same 64-byte chunk (cache line)!
    std::atomic<int> count1{0};
    std::atomic<int> count2{0};
};

void increment1(SharedData& data) {
    for (int i = 0; i < 10'000'000; ++i) {
        data.count1.fetch_add(1, std::memory_order_relaxed);
    }
}

void increment2(SharedData& data) {
    for (int i = 0; i < 10'000'000; ++i) {
        data.count2.fetch_add(1, std::memory_order_relaxed);
    }
}
```
- **Why it is slow**: Thread 1's updates to `count1` force the CPU to invalidate the cache line containing both variables on Thread 2's core, forcing it to reload the line from L3 cache or RAM constantly.
- **The Fix**: Use `alignas(std::hardware_destructive_interference_size)` to separate them onto different cache lines.

---

## 2. Benchmark Invalidation under `-O3`

**The Gotcha**: Compilers compile code under the assumption that if a function has no observable side-effects, it is useless.
- If you call a function inside a benchmark loop but do not print or store the result, the optimizer (`-O3`) can delete the entire function call or loop during compilation.
- Your code will appear to execute in `0 nanoseconds`, invalidating the benchmark.
- **The Fix**: Always use inline assembly compiler barriers or write/sink results to a volatile global variable to prevent elimination.

---

## 3. `std::vector` Reallocation Latency Spikes

Adding an element to a `std::vector` has an **amortized complexity of $O(1)$** (as detailed in the Amortized Complexity section of `[[../00_Complexity_Cheatsheet]]`).
- **The Pitfall**: Although the average cost is $O(1)$, when the vector exceeds its current capacity, it must allocate a larger block of memory, copy or move all existing elements to the new block, and delete the old block.
- **Latency Spikes**: In real-time systems (e.g. trading platforms), this reallocation causes a massive latency spike ($O(N)$ overhead) that can lead to missed SLA deadlines.
- **The Mitigation**: Always call `.reserve(N)` in advance to pre-allocate capacity if the maximum size is known, or use static arrays (`std::array`) to avoid dynamic allocations entirely.

---

## Recap of Tricky Performance Gotchas

| Gotcha | Theme |
|---|---|
| **False Sharing** | Atomic updates to variables on the same cache line force constant cache invalidations; resolve with `alignas`. |
| **Benchmark Invalidation**| Optimizers delete loops lacking side-effects, resulting in false 0ns measurements. |
| **Vector Reallocation** | Average push cost is $O(1)$ amortized, but capacity exhaustion triggers $O(N)$ copy latency spikes. |

---

*<- [[09_Perf_Solutions|Solutions]] · [[../10_Low_Latency_and_Quant_CPP/10_LLQ_Index|Low-Latency & Quant C++ ->]]*
