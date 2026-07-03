---
tags: [dsa, recursion, problems, tower-of-hanoi]
links: ["[[00_Index]]", "[[Recursion_Patterns]]", "[[../Sorting_Algorithms/00_Index]]"]
---

# Recursion -- Problems & Exercises

*<- [[Recursion_Patterns|Patterns]] · [[../Sorting_Algorithms/00_Index|Sorting ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Print 1 to N | Easy | Head recursion | [GFG](https://www.geeksforgeeks.org/print-1-to-n-without-loops/) |
| 2 | Print N to 1 | Easy | Tail recursion | [GFG](https://www.geeksforgeeks.org/print-n-to-1-without-loop/) |
| 3 | Reverse a String | Easy | Swapping indices recursively | [LC 344](https://leetcode.com/problems/reverse-string/) |

## Tier 2 -- Core Variations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 4 | Power of Three | Easy | Division reduction base case | [LC 326](https://leetcode.com/problems/power-of-three/) |
| 5 | Tower of Hanoi | Medium | 3-peg disk transfers | [GFG](https://www.geeksforgeeks.org/c-program-for-tower-of-hanoi/) |
| 6 | K-th Symbol in Grammar | Medium | Parent bit derivation recursion | [LC 779](https://leetcode.com/problems/k-th-symbol-in-grammar/) |

## Tier 3 -- Advanced Recursion

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 7 | Predict the Winner (Recursion) | Medium | Mini-max choices | [LC 486](https://leetcode.com/problems/predict-the-winner/) |
| 8 | All Unique Permutations | Medium | Backtracking recursion | [LC 47](https://leetcode.com/problems/permutations-ii/) |

---

## Worked Solution: Tower of Hanoi

**Key Insight**: We have $n$ disks on peg $A$. We want to move all disks to peg $C$ using peg $B$ as auxiliary, satisfying:
1. Only one disk can be moved at a time.
2. A larger disk cannot be placed on top of a smaller disk.

### Recursive Decomposition
To move $n$ disks from $A \to C$ using $B$:
1. Move top $n-1$ disks from $A \to B$ (using $C$ as auxiliary).
2. Move the largest disk $n$ directly from $A \to C$.
3. Move the $n-1$ disks from $B \to C$ (using $A$ as auxiliary).

- **Recurrence Relation for Moves**: $T(n) = 2T(n-1) + 1 \implies T(n) = 2^n - 1$ moves.

```cpp
#include <iostream>

using namespace std;

// Move n disks from 'fromPeg' to 'toPeg' using 'auxPeg'
void towerOfHanoi(int n, char fromPeg, char toPeg, char auxPeg) {
    if (n == 0) return; // Base case

    // Step 1: Move n-1 disks from source to aux
    towerOfHanoi(n - 1, fromPeg, auxPeg, toPeg);

    // Step 2: Move the largest disk from source to destination
    cout << "Move disk " << n << " from " << fromPeg << " to " << toPeg << "\n";

    // Step 3: Move n-1 disks from aux to destination
    towerOfHanoi(n - 1, auxPeg, toPeg, fromPeg);
}
// Time Complexity: O(2^n)
// Space Complexity: O(n) recursion stack height
```

---

*<- [[Recursion_Patterns|Patterns]] · [[../Sorting_Algorithms/00_Index|Sorting ->]]*
