---
tags: [dsa, dp, stocks, single-transaction]
links: ["[[05_DP_on_Stocks_Index]]", "[[DP_on_Stocks_Infinite_Transactions]]"]
---

# DP on Stocks -- Single Transaction (LC 121)

*<- [[05_DP_on_Stocks_Index|Stocks Index]] · [[DP_on_Stocks_Infinite_Transactions|Infinite Transactions ->]]*

---

## The Problem
You are given an array `prices` where `prices[i]` is the price of a given stock on the $i$-th day. You want to maximize your profit by choosing a single day to buy one stock and choosing a different day in the future to sell that stock.
- Return the maximum profit. If you cannot achieve any profit, return 0.

---

## DP State Formulation & Greedy Intuition
- To maximize profit with only **one** transaction:
  - We want to sell at a high price after buying at a low price.
  - As we iterate through prices, we can maintain the **minimum price seen so far** (`minPrice`).
  - If we sell on the current day `i`, our profit would be `prices[i] - minPrice`.
  - We update our maximum profit if this value is larger than the maximum profit seen so far.
- **State variables**: `minPrice` (minimum buy price), `maxProfit`.

```cpp
#include <vector>
#include <algorithm>

int maxProfit(vector<int>& prices) {
    int n = prices.size();
    if (n == 0) return 0;

    int minPrice = prices[0];
    int maxProfit = 0;

    for (int i = 1; i < n; i++) {
        // Option 1: Current price is lower than minPrice -> update minPrice
        minPrice = min(minPrice, prices[i]);
        
        // Option 2: Sell today -> calculate profit and update maxProfit
        maxProfit = max(maxProfit, prices[i] - minPrice);
    }
    return maxProfit;
}
// Time Complexity: O(n) — single pass
// Space Complexity: O(1)
```

---

## Dry Run Variable State Table:
For `prices = [7, 1, 5, 3, 6, 4]`:
| Iteration (i) | Price | `minPrice` | `prices[i] - minPrice` (Potential Profit) | `maxProfit` |
|---|---|---|---|---|
| Initial | - | 7 | - | 0 |
| i = 1 | 1 | 1 | 0 | 0 |
| i = 2 | 5 | 1 | 4 | 4 |
| i = 3 | 3 | 1 | 2 | 4 |
| i = 4 | 6 | 1 | 5 | 5 |
| i = 5 | 4 | 1 | 3 | 5 |
- **Result**: `5` (Buy on day 1 at price 1, sell on day 4 at price 6).

---

*<- [[05_DP_on_Stocks_Index|Stocks Index]] · [[DP_on_Stocks_Infinite_Transactions|Infinite Transactions ->]]*
