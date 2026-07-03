---
tags: [dsa, number-theory, prime-factorization, fermat, modular-inverse, totient]
links: ["[[00_Index]]", "[[Number_Theory_Problems]]"]
---

# Number Theory -- Primes & Modulo Math

*<- [[00_Index|Index]] · [[Number_Theory_Problems|Problems ->]]*

---

## 1. Prime Factorization (Trial Division)

**Concept**: Any integer $N > 1$ can be uniquely represented as a product of prime numbers.
- **Trial Division**: Iterate from $2$ up to $\sqrt{N}$. If $d$ divides $N$, divide $N$ by $d$ repeatedly until it no longer divides, recording $d$ each time.
- If $N > 1$ at the end of the loop, the remaining value of $N$ must be a prime itself.

```cpp
#include <vector>

vector<int> getPrimeFactors(int n) {
    vector<int> factors;
    // Test divisors up to sqrt(n)
    for (int d = 2; d * d <= n; d++) {
        while (n % d == 0) {
            factors.push_back(d);
            n /= d;
        }
    }
    // If remaining n is prime
    if (n > 1) {
        factors.push_back(n);
    }
    return factors;
}
// Time Complexity: O(sqrt(n)) worst case (when n is prime)
// Space Complexity: O(1) auxiliary space
```

---

## 2. Fermat's Little Theorem & Modular Multiplicative Inverse

### Fermat's Little Theorem
If $p$ is a prime number and $a$ is an integer not divisible by $p$, then:
$$a^{p-1} \equiv 1 \pmod p$$

Multiplying both sides by $a^{-1}$ (the modular multiplicative inverse of $a$ modulo $p$):
$$a^{p-2} \equiv a^{-1} \pmod p$$

This gives us an elegant way to compute division under modular arithmetic:
$$\frac{A}{B} \bmod p = (A \times B^{p-2}) \bmod p$$

```cpp
// Helper: Binary Exponentiation (O(log y))
long long power(long long x, long long y, long long p) {
    long long res = 1;
    x = x % p;
    while (y > 0) {
        if (y & 1) res = (res * x) % p;
        y = y >> 1;
        x = (x * x) % p;
    }
    return res;
}

// Modular Multiplicative Inverse of a modulo prime p
long long modInverse(long long a, long long p) {
    return power(a, p - 2, p);
}
// Time Complexity: O(log p)
// Space Complexity: O(1)
```

---

## 3. Euler's Totient Function $\phi(N)$

**Concept**: $\phi(N)$ counts the number of positive integers up to $N$ that are coprime (relatively prime) to $N$ (i.e. $\gcd(x, N) == 1$).
- **Formula**:
  $$\phi(N) = N \prod_{p | N} \left( 1 - \frac{1}{p} \right)$$
  Where the product is over all distinct prime factors $p$ of $N$.
- **Algorithm**: Find prime factors of $N$ using trial division. Every time we find a prime factor $p$, update:
  $$\text{result} = \text{result} - \frac{\text{result}}{p}$$

```cpp
int phi(int n) {
    int result = n;
    for (int p = 2; p * p <= n; p++) {
        if (n % p == 0) {
            // p is a prime divisor of n
            while (n % p == 0) {
                n /= p;
            }
            result -= result / p;
        }
    }
    // If remaining n is prime
    if (n > 1) {
        result -= result / n;
    }
    return result;
}
// Time Complexity: O(sqrt(n))
// Space Complexity: O(1)
```

---

*<- [[00_Index]] · [[Number_Theory_Problems|Problems ->]]*
