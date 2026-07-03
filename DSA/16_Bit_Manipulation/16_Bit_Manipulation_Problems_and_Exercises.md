---
tags: [dsa, bit-manipulation, problems, exercises]
links: ["[[16_Bit_Manipulation_Index]]", "[[16_Bit_Manipulation_Patterns]]", "[[16_Bit_Manipulation_Tricky]]"]
---

# Bit Manipulation -- Problems & Exercises

*<- [[16_Bit_Manipulation_Patterns|Patterns]] · [[16_Bit_Manipulation_Tricky|Tricky ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Number of 1 Bits | Easy | Brian Kernighan's bit clearing | [LC 191](https://leetcode.com/problems/number-of-1-bits/) |
| 2 | Counting Bits | Easy | DP bit-shift recurrence | [LC 338](https://leetcode.com/problems/counting-bits/) |
| 3 | Single Number | Easy | XOR self-cancellation | [LC 136](https://leetcode.com/problems/single-number/) |

## Tier 2 -- Bit Checking & Shifts

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 4 | Reverse Bits | Easy | Left-right bit shifting | [LC 190](https://leetcode.com/problems/reverse-bits/) |
| 5 | Missing Number | Easy | XOR index vs value comparison | [LC 268](https://leetcode.com/problems/missing-number/) |
| 6 | Subsets (Bitmask variant) | Medium | Binary mask subset enumeration | [LC 78](https://leetcode.com/problems/subsets/) |

## Tier 3 -- Advanced Bitwise Operations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 7 | Single Number II | Medium | Bit position modulo 3 summation | [LC 137](https://leetcode.com/problems/single-number-ii/) |
| 8 | Single Number III | Medium | Split by lowest set bit of XOR difference | [LC 260](https://leetcode.com/problems/single-number-iii/) |
| 9 | Bitwise AND of Numbers Range | Medium | Common prefix of boundaries | [LC 201](https://leetcode.com/problems/bitwise-and-of-numbers-range/) |

## Tier 4 -- Placement-Hard / OA-Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 10 | Sum of Two Integers | Medium | Bitwise carry and sum (no arithmetic) | [LC 371](https://leetcode.com/problems/sum-of-two-integers/) |
| 11 | Maximum Product of Word Lengths | Medium | Bitmask representation of string letters | [LC 318](https://leetcode.com/problems/maximum-product-of-word-lengths/) |

---

## Worked Solution 1: LC 190 -- Reverse Bits

**Key Insight**: We want to reverse bits of a 32-bit unsigned integer.
- Iterate 32 times.
- Shift our `result` left by 1 (to make space).
- Check the LSB of `n` using `n & 1`. OR this bit into the `result`.
- Shift `n` right by 1 to process the next bit.

```cpp
#include <cstdint>

uint32_t reverseBits(uint32_t n) {
    uint32_t result = 0;
    for (int i = 0; i < 32; i++) {
        result = (result << 1) | (n & 1);
        n >>= 1;
    }
    return result;
}
// Time Complexity: O(1) (fixed 32 iterations)
// Space Complexity: O(1)
```

---

## Worked Solution 2: LC 268 -- Missing Number

**Key Insight**: Given an array containing $n$ distinct numbers in the range `[0, n]`, find the one that is missing.
- **Why XOR works**: If we XOR all indices `[0, n]` and all array values, all numbers present in the array will cancel out with their matching indices, leaving only the missing index!
- E.g. indices: `0 ^ 1 ^ 2 ^ 3`, values: `0 ^ 1 ^ 3`.
- Total XOR: `(0^0) ^ (1^1) ^ (3^3) ^ 2 = 0 ^ 0 ^ 0 ^ 2 = 2` (missing number).

```cpp
#include <vector>

int missingNumber(vector<int>& nums) {
    int n = nums.size();
    int xorSum = n; // start with index n

    for (int i = 0; i < n; i++) {
        xorSum ^= i ^ nums[i]; // XOR both index and value
    }
    return xorSum;
}
// Time Complexity: O(n)
// Space Complexity: O(1)
```

---

*<- [[16_Bit_Manipulation_Patterns|Patterns]] · [[16_Bit_Manipulation_Tricky|Tricky ->]]*
