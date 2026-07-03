---
tags: [dsa, bit-manipulation, patterns, xor, kernighan]
links: ["[[16_Bit_Manipulation_Index]]", "[[16_Bit_Manipulation_Problems_and_Exercises]]"]
---

# Bit Manipulation -- Patterns

*<- [[16_Bit_Manipulation_Index|Index]] · [[16_Bit_Manipulation_Problems_and_Exercises|Problems ->]]*

---

## Standard Bitwise Operators Cheatsheet

| Operator | Symbol | Description | Example |
|---|---|---|---|
| **AND** | `&` | 1 if both bits are 1. | `5 & 3` (0101 & 0011 = 0001 = 1) |
| **OR** | `│` | 1 if either bit is 1. | `5 │ 3` (0101 │ 0011 = 0111 = 7) |
| **XOR** | `^` | 1 if bits differ. | `5 ^ 3` (0101 ^ 0011 = 0110 = 6) |
| **NOT** | `~` | Inverts all bits. | `~5` (inverts 00000101 to 11111010) |
| **Left Shift** | `<<` | Shifts bits left, inserts 0 on right. Multiply by 2. | `5 << 1` (0101 -> 1010 = 10) |
| **Right Shift** | `>>` | Shifts bits right, drops rightmost. Divide by 2. | `5 >> 1` (0101 -> 0010 = 2) |

---

## Properties of XOR (`^`) -- The Magic Operator
XOR behaves like addition without carry:
1. **Self-Cancellation**: $x \oplus x = 0$
2. **Identity**: $x \oplus 0 = x$
3. **Commutativity**: $a \oplus b = b \oplus a$
4. **Associativity**: $(a \oplus b) \oplus c = a \oplus (b \oplus c)$

---

## Pattern 1: Single Number (XOR Accumulation -- LC 136)

**The Problem**: Given a non-empty array of integers `nums`, every element appears twice except for one. Find that single one.

### Greedy XOR Choice
By XORing all elements together, all duplicate pairs will cancel out to `0` ($x \oplus x = 0$), leaving only the single unique number ($y \oplus 0 = y$).

```cpp
#include <vector>

int singleNumber(vector<int>& nums) {
    int xorSum = 0;
    for (int x : nums) {
        xorSum ^= x;
    }
    return xorSum;
}
// Time Complexity: O(n)
// Space Complexity: O(1)
```

---

## Pattern 2: Hamming Weight (Brian Kernighan's Algorithm -- LC 191)

**The Problem**: Write a function that takes the binary representation of an unsigned integer and returns the number of '1' bits it has (Hamming Weight).

### Why the Naive Shift is Suboptimal
Shifting bit-by-bit takes 32 operations regardless of how sparse the bits are.

### Brian Kernighan's Algorithm ($O(\text{set\_bits})$)
Doing **`n & (n - 1)`** clears the **lowest set bit** of `n` to `0` in a single operation.
- **Why it works**: Subtracting 1 from `n` flips all bits from the rightmost set bit to the end. Performing a bitwise AND between `n` and `n-1` will keep all other bits unchanged, but zero out the rightmost set bit.
  - Let $n = 12$ (`1100`).
  - $n - 1 = 11$ (`1011`).
  - $n \ \& \ (n-1) = 1100 \ \& \ 1011 = 1000$ (cleared the lowest 1 at index 2).

```cpp
int hammingWeight(uint32_t n) {
    int count = 0;
    while (n > 0) {
        n &= (n - 1); // clear the lowest set bit
        count++;
    }
    return count;
}
// Time Complexity: O(number of 1 bits) -> at most 32 operations, average much less!
// Space Complexity: O(1)
```

---

## Pattern 3: Counting Bits (DP on Bits -- LC 338)

**The Problem**: Given an integer `n`, return an array of length `n + 1` such that for each `i` (`0 <= i <= n`), `ans[i]` is the number of `1` bits in the binary representation of `i`.

### DP Recurrence on Bits
The number of set bits in `i` is directly related to the number of set bits in `i >> 1` (which is `i/2`):
- `i >> 1` is simply `i` shifted right by 1 bit (removing the LSB).
- If the removed LSB is a `1` (which is true if `i` is odd: `i & 1 == 1`), we add 1.
- **Recurrence Relation**: `dp[i] = dp[i >> 1] + (i & 1)`

```cpp
#include <vector>

vector<int> countBits(int n) {
    vector<int> dp(n + 1, 0);
    for (int i = 1; i <= n; i++) {
        dp[i] = dp[i >> 1] + (i & 1);
    }
    return dp;
}
// Time Complexity: O(n) — single pass, O(1) per number
// Space Complexity: O(1) (excluding output array)
```

### Dry Run Variable State Table:
For `n = 5`:
| Number (i) | Binary | `i >> 1` (Index) | `dp[i >> 1]` | `i & 1` (LSB) | `dp[i]` |
|---|---|---|---|---|---|
| 0 | `0000` | - | - | - | 0 |
| 1 | `0001` | 0 | 0 | 1 | 0 + 1 = 1 |
| 2 | `0010` | 1 | 1 | 0 | 1 + 0 = 1 |
| 3 | `0011` | 1 | 1 | 1 | 1 + 1 = 2 |
| 4 | `0100` | 2 | 1 | 0 | 1 + 0 = 1 |
| 5 | `0101` | 2 | 1 | 1 | 1 + 1 = 2 |
- **Result**: `[0, 1, 1, 2, 1, 2]` (Correct).

---

*<- [[16_Bit_Manipulation_Index|Index]] · [[16_Bit_Manipulation_Problems_and_Exercises|Problems ->]]*
