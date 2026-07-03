---
tags: [dsa, math, geometry, sieve, gcd, matrix-rotation]
links: ["[[17_Math_and_Geometry_Index]]", "[[17_Math_and_Geometry_Problems_and_Exercises]]"]
---

# Math & Geometry -- Patterns

*<- [[17_Math_and_Geometry_Index|Index]] · [[17_Math_and_Geometry_Problems_and_Exercises|Problems ->]]*

---

## Pattern 1: Sieve of Eratosthenes (Prime Generation -- LC 204)

**The Problem**: Count the number of prime numbers less than a non-negative number $n$.

### Intuition
Instead of testing each number for primality in $O(\sqrt{n})$ time (total $O(n \sqrt{n})$), start from $2$ and mark all of its multiples as composite. The next un-marked number is guaranteed to be prime.
- **Optimization**: For a prime $p$, we can start marking its multiples from $p^2$ (since smaller multiples like $2p, 3p$ would have already been marked by smaller primes).
- **Time Complexity**: $O(n \log \log n)$ (sum of reciprocals of primes converges to $\log \log n$).
- **Space Complexity**: O(n) boolean array.

```cpp
#include <vector>

int countPrimes(int n) {
    if (n <= 2) return 0;

    vector<bool> isPrime(n, true);
    isPrime[0] = isPrime[1] = false;

    // We only need to loop up to sqrt(n)
    for (int p = 2; p * p < n; p++) {
        if (isPrime[p]) {
            // Mark all multiples starting from p*p
            for (int i = p * p; i < n; i += p) {
                isPrime[i] = false;
            }
        }
    }

    int primeCount = 0;
    for (int i = 2; i < n; i++) {
        if (isPrime[i]) primeCount++;
    }
    return primeCount;
}
```

---

## Pattern 2: Greatest Common Divisor (GCD) (Euclidean Algorithm)

**The Problem**: Find the greatest common divisor of two integers `a` and `b`.

### The Euclidean Theorem
$$\gcd(a, b) = \gcd(b, a \bmod b)$$
This division-based reduction shrinks values exponentially.
- **Time Complexity**: $O(\log(\min(a, b)))$ (worst case is consecutive Fibonacci numbers).
- **Space Complexity**: $O(\log(\min(a, b)))$ recursion stack.

```cpp
int gcd(int a, int b) {
    if (b == 0) return a;
    return gcd(b, a % b);
}

// Iterative (O(1) Space)
int gcdIterative(int a, int b) {
    while (b != 0) {
        int temp = b;
        b = a % b;
        a = temp;
    }
    return a;
}
```

---

## Pattern 3: Rotate Matrix 90 Degrees Clockwise (LC 48)

**The Problem**: Rotate an $n \times n$ 2D matrix 90 degrees clockwise in-place.

### Elegant 2-Step Geometric Transformation
Instead of complex index swapping arithmetic, we can perform two simple matrix operations:
1. **Transpose the matrix**: Swap `matrix[r][c]` with `matrix[c][r]`. This swaps rows with columns along the main diagonal.
2. **Reverse each row**: Reversing the columns of each row yields the 90-degree rotated matrix!

```
Original:             Transpose:            Reverse Rows (Final):
  1  2  3               1  4  7               7  4  1
  4  5  6     ──►       2  5  8     ──►       8  5  2
  7  8  9               3  6  9               9  6  3
```

```cpp
#include <vector>
#include <algorithm>

void rotate(vector<vector<int>>& matrix) {
    int n = matrix.size();

    // Step 1: Transpose in-place
    for (int r = 0; r < n; r++) {
        for (int c = r + 1; c < n; c++) {
            swap(matrix[r][c], matrix[c][r]);
        }
    }

    // Step 2: Reverse each row
    for (int r = 0; r < n; r++) {
        reverse(matrix[r].begin(), matrix[r].end());
    }
}
// Time Complexity: O(n^2) — visit each cell twice
// Space Complexity: O(1) in-place
```

---

## Pattern 4: Happy Number (Cycle Detection -- LC 202)

**The Problem**: Determine if a number $n$ is happy. A happy number is defined by replacing the number by the sum of the squares of its digits, repeating until it equals 1, or looping endlessly in a cycle.

### Fast/Slow Pointer Cycle Detection
This is equivalent to detecting a cycle in a linked list. We can use **Floyd's Cycle Finding Algorithm** (one pointer moves 1 step, the other moves 2 steps). If they meet at a value other than 1, a cycle exists (not happy).

```cpp
int getNextSum(int n) {
    int sum = 0;
    while (n > 0) {
        int d = n % 10;
        sum += d * d;
        n /= 10;
    }
    return sum;
}

bool isHappy(int n) {
    int slow = n;
    int fast = getNextSum(n);

    while (fast != 1 && slow != fast) {
        slow = getNextSum(slow);             // 1 step
        fast = getNextSum(getNextSum(fast)); // 2 steps
    }
    return fast == 1;
}
// Time Complexity: O(log n) average digits reduction
// Space Complexity: O(1)
```

---

*<- [[17_Math_and_Geometry_Index|Index]] · [[17_Math_and_Geometry_Problems_and_Exercises|Problems ->]]*
