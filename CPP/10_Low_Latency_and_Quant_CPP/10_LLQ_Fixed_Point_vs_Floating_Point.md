---
tags: [cpp, low-latency, quant, floating-point, fixed-point, IEEE-754]
links: ["[[10_LLQ_Index]]", "[[10_LLQ_Avoiding_Allocations_in_Hot_Path]]", "[[10_LLQ_NUMA_and_Kernel_Bypass_Concepts]]"]
---

# Low-Latency & Quant C++ -- Fixed-Point vs Floating-Point

*<- [[10_LLQ_Avoiding_Allocations_in_Hot_Path|Avoiding Allocations in Hot Path]] · [[10_LLQ_NUMA_and_Kernel_Bypass_Concepts|NUMA & Kernel Bypass Concepts ->]]*

---

Floating-point numbers (`float`, `double`) represent values using a binary fraction (IEEE-754). This introduces precision limitations and non-determinism that are unacceptable in high-frequency trading (HFT) matching engines or settlement systems.

---

## 1. The Precision & Determinism Problem

### A. Non-exact base-10 representations
Fractions like `0.1` or `0.2` have infinite repeating fractions in binary:
- `0.1 + 0.2` does not exactly equal `0.3` (it yields `0.30000000000000004` in double precision).
- Rounding errors accumulate over millions of trades, creating financial discrepancies.

### B. Hardware & Optimization Non-Determinism
Different compilers, CPU architectures, or optimization flags (like Fused Multiply-Add instruction options) can evaluate the same floating-point operations differently. 
A trade matching engine running on Intel might get a slightly different price than the same engine running on AMD, causing database reconciliation failures.

---

## 2. Fixed-Point Arithmetic (The Solution)

**Fixed-Point** arithmetic represents decimals by scaling values and storing them as **integers**.
- For example, if we need 4 decimal places of precision, the value `$100.25` is stored as the integer `1002500` (scaled by $10,000$).
- All calculations are performed using pure integer arithmetic, guaranteeing absolute precision, consistency, and execution speed.

```cpp
#include <iostream>
#include <cstdint>

class FixedPoint4D {
    std::int64_t rawValue = 0;
    static constexpr std::int64_t MULTIPLIER = 10000; // 4 decimal places

    // Private constructor from raw integer representation
    explicit FixedPoint4D(std::int64_t rawVal, bool) : rawValue(rawVal) {}

public:
    FixedPoint4D() = default;
    
    // Construct from double (used at borders, not in hot loop)
    explicit FixedPoint4D(double val) {
        rawValue = static_cast<std::int64_t>(val * MULTIPLIER + (val >= 0 ? 0.5 : -0.5));
    }

    FixedPoint4D operator+(const FixedPoint4D& other) const {
        return FixedPoint4D(rawValue + other.rawValue, true);
    }

    FixedPoint4D operator-(const FixedPoint4D& other) const {
        return FixedPoint4D(rawValue - other.rawValue, true);
    }

    FixedPoint4D operator*(const FixedPoint4D& other) const {
        // (A / M) * (B / M) * M = (A * B) / M
        return FixedPoint4D((rawValue * other.rawValue) / MULTIPLIER, true);
    }

    double toDouble() const {
        return static_cast<double>(rawValue) / MULTIPLIER;
    }

    void print() const {
        std::cout << rawValue / MULTIPLIER << "." 
                  << std::abs(rawValue % MULTIPLIER) << "\n";
    }
};

void demoFixedPoint() {
    // Floating point discrepancy:
    double f1 = 0.1;
    double f2 = 0.2;
    std::cout << std::boolalpha << (f1 + f2 == 0.3) << "\n"; // Outputs: false!

    // Fixed point correctness:
    FixedPoint4D p1(0.1);
    FixedPoint4D p2(0.2);
    FixedPoint4D p3 = p1 + p2;
    std::cout << (p3.toDouble() == 0.3) << "\n"; // Outputs: true! (Deterministic!)
}
```

---

*<- [[10_LLQ_Avoiding_Allocations_in_Hot_Path|Avoiding Allocations in Hot Path]] · [[10_LLQ_NUMA_and_Kernel_Bypass_Concepts|NUMA & Kernel Bypass Concepts ->]]*
