---
tags: [dsa, number-theory, problems, modular-exponentiation]
links: ["[[00_Index]]", "[[Number_Theory_Patterns]]", "[[../../00_Home]]"]
---

# Number Theory -- Problems & Exercises

*<- [[Number_Theory_Patterns|Prime Math]] · [[../../00_Home|Home ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Prime Factorization | Easy | Trial division up to $\sqrt{N}$ | [GFG](https://www.geeksforgeeks.org/print-all-prime-factors-of-a-given-number/) |
| 2 | Euler Totient Function | Medium | $\phi(N)$ prime factor accumulation | [GFG](https://www.geeksforgeeks.org/eulers-totient-function/) |

## Tier 2 -- Modulo Exponentiation

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 3 | Super Pow | Medium | Modulo arithmetic on array exponent | [LC 372](https://leetcode.com/problems/super-pow/) |
| 4 | Greatest Common Divisor | Easy | Euclidean algorithm division | [GFG](https://www.geeksforgeeks.org/c-program-find-gcd-two-numbers/) |

## Tier 3 -- Combinatorics & Hard Modulo

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 5 | Find Unique BSTs (Catalan) | Medium | Modular inverse for combination quotients | [LC 96](https://leetcode.com/problems/unique-binary-search-trees/) |

---

## Worked Solution: LC 372 -- Super Pow

**Key Insight**: Calculate $a^b \bmod 1337$ where $b$ is an array of digits representing the exponent.
- The modulus $q = 1337$ is composite ($1337 = 7 \times 191$).
- Using properties of exponents:
  $$a^{[d_1, d_2, ..., d_k]} = \left( a^{[d_1, ..., d_{k-1}]} \right)^{10} \times a^{d_k}$$
- This allows a clean recursive formulation for digits:
  `superPow(a, b) = (pow(superPow(a, b_cropped), 10) * pow(a, b_last)) % 1337`.

```cpp
#include <vector>

using namespace std;

class Solution {
    const int MOD = 1337;

    // Helper: Modulo Binary Exponentiation
    int power(int x, int y) {
        x %= MOD;
        int res = 1;
        while (y > 0) {
            if (y & 1) res = (res * x) % MOD;
            y >>= 1;
            x = (x * x) % MOD;
        }
        return res;
    }

public:
    int superPow(int a, vector<int>& b) {
        if (b.empty()) return 1;

        int lastDigit = b.back();
        b.pop_back(); // crop last digit

        // Part 1: (superPow(a, cropped_b))^10 % MOD
        int part1 = power(superPow(a, b), 10);
        // Part 2: a^lastDigit % MOD
        int part2 = power(a, lastDigit);

        return (part1 * part2) % MOD;
    }
};
// Time Complexity: O(Length of b) — loops/recurses exactly digit counts
// Space Complexity: O(Length of b) recursion stack
```

---

*<- [[Number_Theory_Patterns|Prime Math]] · [[../../00_Home|Home ->]]*
