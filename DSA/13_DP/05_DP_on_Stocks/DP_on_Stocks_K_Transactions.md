---
tags: [dsa, dp, stocks, k-transactions]
links: ["[[05_DP_on_Stocks_Index]]", "[[DP_on_Stocks_Two_Transactions]]", "[[DP_on_Stocks_Problems]]"]
---

# DP on Stocks -- K Transactions (LC 188)

*<- [[DP_on_Stocks_Two_Transactions|Two Transactions]] · [[DP_on_Stocks_Problems|Problems ->]]*

---

## The Problem
You are given an integer array `prices` where `prices[i]` is the price of a given stock on the $i$-th day, and an integer `k`. Find the maximum profit you can achieve. You may complete **at most k transactions**.
- Note: You may not engage in multiple transactions simultaneously.

---

## 3D DP State Formulation

Let `dp[i][t][hold]` be the maximum profit on day `i`, with `t` transactions completed, and `hold` state (0 if not holding, 1 if holding).

At day `i` (price `p = prices[i]`):
1. **Not Holding (`hold = 0`)**:
   - Either continue not holding: `dp[i-1][t][0]`.
   - Or sell the stock today: `dp[i-1][t][1] + p`.
   - **Formula**: `dp[i][t][0] = max(dp[i-1][t][0], dp[i-1][t][1] + p)`
2. **Holding (`hold = 1`)**:
   - Either continue holding: `dp[i-1][t][1]`.
   - Or buy the stock today (which increases transaction count `t` from `t-1` to `t`): `dp[i-1][t-1][0] - p`.
   - **Formula**: `dp[i][t][1] = max(dp[i-1][t][1], dp[i-1][t-1][0] - p)`

---

## Space-Optimized Implementation (2D Arrays)
- Since day `i` only reads from day `i-1`, we can eliminate the day dimension `i` completely, maintaining only `prev[t][hold]`.
- **Optimization (if $k \ge n/2$)**: If $k$ is greater than half the number of days, the transaction limit is effectively infinite (we can't make more than $n/2$ transactions anyway because a transaction requires at least 2 days: buy on day A, sell on day B). We can bypass DP and run the simple $O(n)$ greedy infinite transaction solver (LC 122) to avoid memory overhead.

```cpp
#include <vector>
#include <algorithm>

// Greedy solver for infinite transactions (from LC 122)
int maxProfitInfinite(const vector<int>& prices) {
    int maxProfit = 0;
    for (int i = 1; i < (int)prices.size(); i++) {
        if (prices[i] > prices[i - 1]) {
            maxProfit += prices[i] - prices[i - 1];
        }
    }
    return maxProfit;
}

int maxProfitKTransactions(int k, vector<int>& prices) {
    int n = prices.size();
    if (n <= 1 || k == 0) return 0;

    // Optimization check
    if (k >= n / 2) return maxProfitInfinite(prices);

    // dp[t][hold]: t goes from 0 to k, hold is 0 or 1
    // Initialize holding values to -1e9 (negative infinity)
    vector<vector<int>> dp(k + 1, vector<int>(2, 0));
    for (int t = 0; t <= k; t++) {
        dp[t][1] = -prices[0]; // base case buy on day 0
    }

    for (int i = 1; i < n; i++) {
        int p = prices[i];
        for (int t = 1; t <= k; t++) {
            dp[t][0] = max(dp[t][0], dp[t][1] + p);       // Sell
            dp[t][1] = max(dp[t][1], dp[t - 1][0] - p);   // Buy (uses t-1 free state)
        }
    }
    return dp[k][0];
}
// Time Complexity: O(n * k)
// Space Complexity: O(k)
```

---

*<- [[DP_on_Stocks_Two_Transactions|Two Transactions]] · [[DP_on_Stocks_Problems|Problems ->]]*
