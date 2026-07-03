---
tags: [dsa, dp, stocks, two-transactions]
links: ["[[05_DP_on_Stocks_Index]]", "[[DP_on_Stocks_With_Fee]]", "[[DP_on_Stocks_K_Transactions]]"]
---

# DP on Stocks -- Two Transactions (LC 123)

*<- [[DP_on_Stocks_With_Fee|With Transaction Fee]] · [[DP_on_Stocks_K_Transactions|K Transactions ->]]*

---

## The Problem
You are given an array `prices` where `prices[i]` is the price of a given stock on the $i$-th day. Find the maximum profit you can achieve. You may complete **at most two transactions**.
- Note: You may not engage in multiple transactions simultaneously.

---

## State Expansion Formulation
Since we can execute at most 2 transactions, we can define the problem as progressing through **four sequential transaction states** as we iterate day-by-day:

1. **`firstBuy`**: Net profit after buying the 1st stock. We want to buy at the lowest price possible (maximize `-price`).
   `firstBuy = max(firstBuy, -price)`
2. **`firstSell`**: Net profit after selling the 1st stock. We add current price to our `firstBuy` state.
   `firstSell = max(firstSell, firstBuy + price)`
3. **`secondBuy`**: Net profit after buying the 2nd stock. We deduct current price from our `firstSell` state.
   `secondBuy = max(secondBuy, firstSell - price)`
4. **`secondSell`**: Net profit after selling the 2nd stock. We add current price to our `secondBuy` state.
   `secondSell = max(secondSell, secondBuy + price)`

- **Base Cases (Day 0)**:
  - `firstBuy = -prices[0]`
  - `firstSell = 0`
  - `secondBuy = -prices[0]` (buying the second stock immediately after selling it at price 0 profit)
  - `secondSell = 0`

```cpp
#include <vector>
#include <algorithm>

int maxProfitTwoTransactions(vector<int>& prices) {
    int n = prices.size();
    if (n == 0) return 0;

    int firstBuy = -prices[0];
    int firstSell = 0;
    int secondBuy = -prices[0];
    int secondSell = 0;

    for (int i = 1; i < n; i++) {
        int p = prices[i];
        
        firstBuy   = max(firstBuy, -p);
        firstSell  = max(firstSell, firstBuy + p);
        secondBuy  = max(secondBuy, firstSell - p);
        secondSell = max(secondSell, secondBuy + p);
    }
    return secondSell; // final maximized profit after at most 2 transactions
}
// Time Complexity: O(n)
// Space Complexity: O(1)
```

---

## Dry Run Variable State Table:
For `prices = [3, 3, 5, 0, 0, 3, 1, 4]`:
| Day (i) | Price | `firstBuy` | `firstSell` | `secondBuy` | `secondSell` |
|---|---|---|---|---|---|
| 0 | 3 | -3 | 0 | -3 | 0 |
| 1 | 3 | -3 | 0 | -3 | 0 |
| 2 | 5 | -3 | 2 | -3 | 2 |
| 3 | 0 | 0 | 2 | 2 | 2 |
| 4 | 0 | 0 | 2 | 2 | 2 |
| 5 | 3 | 0 | 3 | 2 | 5 |
| 6 | 1 | 0 | 3 | 2 | 5 |
| 7 | 4 | 0 | 4 | 2 | 6 |
- **Result**: `6` (Correct: Buy at 3, sell at 5. Buy at 0, sell at 4. Total profit = 2 + 4 = 6).

---

*<- [[DP_on_Stocks_With_Fee|With Transaction Fee]] · [[DP_on_Stocks_K_Transactions|K Transactions ->]]*
