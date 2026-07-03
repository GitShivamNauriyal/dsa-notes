---
tags: [dsa, dp, 1d-dp, coin-change, min-cost-climbing-stairs]
links: ["[[01_1D_DP_Index]]", "[[1D_DP_Fibonacci_Pattern]]"]
---

# 1D DP -- Coin Change & Step Minimums

*<- [[1D_DP_Fibonacci_Pattern|Fibonacci & Step Patterns]] · [[1D_DP_Problems|Problems ->]]*

---

## 1. Min Cost Climbing Stairs (LC 746)

**The Problem**: You are given an integer array `cost` where `cost[i]` is the cost of $i$-th step on a staircase. Once you pay the cost, you can either climb one or two steps. You can either start from the step with index 0, or the step with index 1. Return the minimum cost to reach the top of the floor.

### State Formulation
Let `dp[i]` be the minimum cost to reach step `i`.
- To reach step `i`, you must climb from `i-1` or `i-2`. 
- Paying the cost happens *when leaving* a step. Thus, if you climb from `i-1`, the cost is `dp[i-1] + cost[i-1]`.
- **Recurrence Relation**: `dp[i] = min(dp[i-1] + cost[i-1], dp[i-2] + cost[i-2])`
- **Base cases**: `dp[0] = 0`, `dp[1] = 0` (you can start at either 0 or 1 for free).

```cpp
#include <vector>
#include <algorithm>

int minCostClimbingStairs(vector<int>& cost) {
    int n = cost.size();
    int prev2 = 0; // dp[i-2]
    int prev1 = 0; // dp[i-1]
    int curr = 0;

    for (int i = 2; i <= n; i++) {
        curr = min(prev1 + cost[i - 1], prev2 + cost[i - 2]);
        prev2 = prev1;
        prev1 = curr;
    }
    return curr;
}
// Time Complexity: O(n)
// Space Complexity: O(1)
```

---

## 2. Coin Change (LC 322)

**The Problem**: You are given an integer array `coins` representing coins of different denominations and an integer `amount` representing a total amount of money. Return the fewest number of coins that you need to make up that amount. If that amount of money cannot be made up by any combination of the coins, return -1.
- You may assume that you have an infinite number of each kind of coin.

### A. Trivial Recursion (Top-Down)
- **State Definition**: `solve(amt)` returns the minimum coins required to make sum `amt`.
- **Recurrence**: Check every coin `c` in `coins`. If `amt - c >= 0`, recurse on `solve(amt - c)`.
- **Recurrence Relation**: `solve(amt) = 1 + min(solve(amt - c))` for all `c` in `coins`.
- **Base case**: `solve(0) = 0`, if `amt < 0` return $\infty$.
- **Time Complexity**: $O(c^{\text{amount}})$ where $c$ is the number of coins (exponential, search tree depth is `amount`).

```cpp
#include <vector>
#include <algorithm>

int solveRec(int amt, const vector<int>& coins) {
    if (amt == 0) return 0;
    if (amt < 0) return 1e9; // 1e9 represents infinity

    int minCoins = 1e9;
    for (int c : coins) {
        minCoins = min(minCoins, 1 + solveRec(amt - c, coins));
    }
    return minCoins;
}
```

---

### B. Memoization (Top-Down)
- Cache computed amounts.
- **Time Complexity**: $O(\text{amount} \times c)$ (each amount state evaluated once, looping over $c$ coins).
- **Space Complexity**: $O(\text{amount})$ array + recursion stack.

```cpp
#include <vector>
#include <algorithm>

int solveMemo(int amt, const vector<int>& coins, vector<int>& memo) {
    if (amt == 0) return 0;
    if (amt < 0) return 1e9;
    if (memo[amt] != -2) return memo[amt];

    int minCoins = 1e9;
    for (int c : coins) {
        minCoins = min(minCoins, 1 + solveMemo(amt - c, coins, memo));
    }
    return memo[amt] = minCoins;
}

int coinChangeMemo(vector<int>& coins, int amount) {
    vector<int> memo(amount + 1, -2);
    int res = solveMemo(amount, coins, memo);
    return res >= 1e9 ? -1 : res;
}
```

---

### C. Tabulation (Bottom-Up)
- **DP Table Definition**: `dp[i]` is the minimum coins needed to make amount `i`.
- Fill the table from `1` to `amount`.
- **Time Complexity**: $O(\text{amount} \times c)$.
- **Space Complexity**: $O(\text{amount})$ to store the DP table.

```cpp
#include <vector>
#include <algorithm>

int coinChangeTabulation(vector<int>& coins, int amount) {
    // Fill with amount + 1 (representing infinity, since max coins is amount)
    vector<int> dp(amount + 1, amount + 1);
    dp[0] = 0; // base case: 0 coins needed for amount 0

    for (int i = 1; i <= amount; i++) {
        for (int c : coins) {
            if (i - c >= 0) {
                dp[i] = min(dp[i], 1 + dp[i - c]);
            }
        }
    }
    return dp[amount] > amount ? -1 : dp[amount];
}
```

### Dry Run Variable State Table:
For `coins = [1, 2, 5]`, `amount = 5`:
| Amount (i) | `dp[i - 1]` (coin 1) | `dp[i - 2]` (coin 2) | `dp[i - 5]` (coin 5) | `dp[i]` |
|---|---|---|---|---|
| 0 | - | - | - | 0 |
| 1 | `1 + dp[0] = 1` | Out of Bounds | Out of Bounds | 1 |
| 2 | `1 + dp[1] = 2` | `1 + dp[0] = 1` | Out of Bounds | 1 |
| 3 | `1 + dp[2] = 2` | `1 + dp[1] = 2` | Out of Bounds | 2 |
| 4 | `1 + dp[3] = 3` | `1 + dp[2] = 2` | Out of Bounds | 2 |
| 5 | `1 + dp[4] = 3` | `1 + dp[3] = 3` | `1 + dp[0] = 1` | 1 |
- **Result**: `1` (Correct: Pick one 5-coin).

---

*<- [[1D_DP_Fibonacci_Pattern|Fibonacci & Step Patterns]] · [[1D_DP_Problems|Problems ->]]*
