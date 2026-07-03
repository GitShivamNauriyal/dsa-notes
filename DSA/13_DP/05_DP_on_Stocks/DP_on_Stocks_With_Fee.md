---
tags: [dsa, dp, stocks, transaction-fee]
links: ["[[05_DP_on_Stocks_Index]]", "[[DP_on_Stocks_With_Cooldown]]", "[[DP_on_Stocks_Two_Transactions]]"]
---

# DP on Stocks -- With Transaction Fee (LC 714)

*<- [[DP_on_Stocks_With_Cooldown|With Cooldown]] · [[DP_on_Stocks_Two_Transactions|Two Transactions ->]]*

---

## The Problem
You are given an array `prices` where `prices[i]` is the price of a given stock on the $i$-th day, and an integer `fee` representing a transaction fee. Find the maximum profit you can achieve with as many transactions as you want, with the fee paid **exactly once** for each complete transaction.
- Note: You may not engage in multiple transactions simultaneously (i.e., you must sell the stock before you buy again).

---

## State Machine Formulation

We have two states:
1. **`hold`**: Owning a stock.
   - Transition: Either continue holding the stock (`hold`), or buy a new stock today: `free - price`.
   - **Formula**: `hold = max(hold, free - price)`
2. **`free`**: Not owning any stock.
   - Transition: Either stay free of stock (`free`), or sell the stock today: `hold + price - fee`. (Note: We deduct the `fee` when we sell).
   - **Formula**: `free = max(free, hold + price - fee)`

- **Base Cases (Day 0)**:
  - `hold = -prices[0]`
  - `free = 0`

```cpp
#include <vector>
#include <algorithm>

int maxProfitWithFee(vector<int>& prices, int fee) {
    int n = prices.size();
    if (n == 0) return 0;

    int hold = -prices[0];
    int free = 0;

    for (int i = 1; i < n; i++) {
        int nextHold = max(hold, free - prices[i]);
        int nextFree = max(free, hold + prices[i] - fee); // fee deducted on selling
        hold = nextHold;
        free = nextFree;
    }
    return free;
}
// Time Complexity: O(n)
// Space Complexity: O(1)
```

---

## Dry Run Variable State Table:
For `prices = [1, 3, 2, 8, 4, 9]`, `fee = 2`:
| Day (i) | Price | `hold` state | `free` state | Decision Calculations |
|---|---|---|---|---|
| 0 | 1 | -1 | 0 | Base cases |
| 1 | 3 | -1 | 0 | `hold=max(-1, 0-3)=-1`, `free=max(0, -1+3-2)=0` |
| 2 | 2 | -1 | 0 | `hold=max(-1, 0-2)=-1`, `free=max(0, -1+2-2)=0` |
| 3 | 8 | -1 | 5 | `hold=max(-1, 0-8)=-1`, `free=max(0, -1+8-2)=5` |
| 4 | 4 | 1 | 5 | `hold=max(-1, 5-4)=1`,  `free=max(5, -1+4-2)=5` |
| 5 | 9 | 1 | 8 | `hold=max(1, 5-9)=1`,   `free=max(5, 1+9-2)=8` |
- **Result**: `8` (Correct: Buy at 1, sell at 8, buy at 4, sell at 9. Profit = (8-1-2) + (9-4-2) = 5 + 3 = 8).

---

*<- [[DP_on_Stocks_With_Cooldown|With Cooldown]] · [[DP_on_Stocks_Two_Transactions|Two Transactions ->]]*
