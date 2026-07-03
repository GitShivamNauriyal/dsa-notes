---
tags: [dsa, bit-manipulation, tricky, hard, carry-add, partition]
links: ["[[16_Bit_Manipulation_Index]]", "[[16_Bit_Manipulation_Problems_and_Exercises]]", "[[../17_Math_and_Geometry/17_Math_and_Geometry_Index]]"]
---

# Bit Manipulation -- Tricky & Higher-Order

*<- [[16_Bit_Manipulation_Problems_and_Exercises|Problems]] · [[../17_Math_and_Geometry/17_Math_and_Geometry_Index|Math & Geometry ->]]*

---

## 1. Sum of Two Integers (No Arithmetic Operators -- LC 371)

**Why Tricky**: Add two integers `a` and `b` without using `+` or `-`. 
- We must simulate a **binary adder** circuit.

### Binary Adder Logic
For any two binary digits:
- **Sum (without carry)**: $a \oplus b$ (XOR is 1 if bits differ).
- **Carry**: $a \ \& \ b$ (AND is 1 if both are 1). Since carry moves to the next higher bit, we left-shift it by 1: `(a & b) << 1`.

We add the sum and carry recursively until the carry becomes 0.

```cpp
int getSum(int a, int b) {
    while (b != 0) {
        int carry = a & b; // find common set bits
        a = a ^ b;         // sum without carry
        b = (unsigned int)carry << 1; // shift carry to left (cast to prevent negative overflow checks)
    }
    return a;
}
// Time Complexity: O(1) (loops at most 32 times for 32-bit integers)
// Space Complexity: O(1)
```

### Dry Run Variable State Table:
For `a = 5` (`0101`), `b = 3` (`0011`):
| Iteration | `a` | `b` | `a & b` (Carry) | `a ^ b` (Sum) | `carry << 1` (New `b`) |
|---|---|---|---|---|---|
| Initial | `0101` (5) | `0011` (3) | - | - | - |
| Loop 1 | `0101` | `0011` | `0001` | `0110` (6) | `0010` (2) |
| Loop 2 | `0110` | `0010` | `0010` | `0100` (4) | `0100` (4) |
| Loop 3 | `0100` | `0100` | `0100` | `0000` (0) | `1000` (8) |
| Loop 4 | `0000` | `1000` | `0000` | `1000` (8) | `0000` (0) |
- **Result**: `8` (Correct).

---

## 2. Single Number III (Dual Unique Elements -- LC 260)

**Why Tricky**: Every element appears twice except for **two** numbers. Find those two numbers.
- If we XOR all elements: `xorSum = val1 ^ val2` (since duplicates cancel out).
- We cannot extract `val1` and `val2` directly from `xorSum`.

### The Partitioning Trick
1. `xorSum` contains a set bit `1` at any index where the bits of `val1` and `val2` differ.
2. Find the lowest set bit of `xorSum`: `diffBit = xorSum & -xorSum` (using two's complement).
3. Group all numbers in the array into two separate categories:
   - **Group A**: Numbers where `diffBit` is set.
   - **Group B**: Numbers where `diffBit` is NOT set.
4. `val1` and `val2` will end up in different groups because their bits differ at that position.
5. All duplicate pairs will end up in the **same** group (since they are identical).
6. XORing each group separately cancels out the duplicates, leaving `val1` in Group A and `val2` in Group B!

```cpp
#include <vector>

vector<int> singleNumber(vector<int>& nums) {
    long long xorSum = 0;
    for (int x : nums) xorSum ^= x;

    // Get the rightmost set bit
    int diffBit = xorSum & -xorSum;

    int val1 = 0, val2 = 0;
    for (int x : nums) {
        if (x & diffBit) {
            val1 ^= x; // XOR Group A
        } else {
            val2 ^= x; // XOR Group B
        }
    }
    return {val1, val2};
}
// Time Complexity: O(n) — two passes
// Space Complexity: O(1)
```

---

*<- [[16_Bit_Manipulation_Problems_and_Exercises|Problems]] · [[../17_Math_and_Geometry/17_Math_and_Geometry_Index|Math & Geometry ->]]*
