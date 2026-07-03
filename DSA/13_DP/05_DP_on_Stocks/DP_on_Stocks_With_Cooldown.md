---
tags: [dsa, dp, stocks, cooldown]
links: ["[[05_DP_on_Stocks_Index]]", "[[DP_on_Stocks_Infinite_Transactions]]", "[[DP_on_Stocks_With_Fee]]"]
---

# DP on Stocks -- With Cooldown (LC 309)

*<- [[DP_on_Stocks_Infinite_Transactions|Infinite Transactions]] · [[DP_on_Stocks_With_Fee|With Transaction Fee ->]]*

---

## The Problem
You are given an array `prices` where `prices[i]` is the price of a given stock on the $i$-th day. Find the maximum profit you can achieve. You may complete as many transactions as you like (i.e., buy one and sell one share of the stock multiple times) with the following constraints:
- After you sell your stock, you cannot buy stock on the next day (i.e., **cooldown day**).

---

## State Machine Formulation

We have 3 distinct states:
1. **`hold`**: Owning a stock.
   - Transition: Either continue holding (`hold`), or buy a stock today. 
   - **Crucial Constraint**: You can only buy a stock if you were in the `cooldown` state yesterday. You cannot buy if you sold yesterday!
   - **Formula**: `hold = max(hold, cooldown - price)`
2. **`sold`**: Just sold a stock today.
   - Transition: You must have owned a stock yesterday (`hold`), and you sell it today.
   - **Formula**: `sold = hold + price`
3. **`cooldown`**: Not owning any stock, and *not* having sold today (i.e. you are in a cooldown phase or rest phase).
   - Transition: Either you were in the `cooldown` state yesterday and did nothing (`cooldown`), or you just finished a `sold` state yesterday (`sold` from yesterday).
   - **Formula**: `cooldown = max(cooldown, prev_sold)`

```
          Buy
   ┌───────────────┐
   ▼               │
[hold] ──Sell──► [sold] ──Rest──► [cooldown]
  ▲                 │                 ▲
  │   Rest          │                 │   Rest
  └─── (loop)       └─────────────────┘
```

- **Base Cases (Day 0)**:
  - `hold = -prices[0]`
  - `sold = 0`
  - `cooldown = 0`

```cpp
#include <vector>
#include <algorithm>

int maxProfitWithCooldown(vector<int>& prices) {
    int n = prices.size();
    if (n <= 1) return 0;

    int hold = -prices[0];
    int sold = 0;
    int cooldown = 0;

    for (int i = 1; i < n; i++) {
        int prev_sold = sold; // cache yesterday's sold state
        
        hold = max(hold, cooldown - prices[i]);
        sold = hold + prices[i];
        cooldown = max(cooldown, prev_sold);
    }
    // Return max profit of final states where we do not hold a stock
    return max(sold, cooldown);
}
// Time Complexity: O(n)
// Space Complexity: O(1)
```

---

## Dry Run Variable State Table:
For `prices = [1, 2, 3, 0, 2]`:
| Day (i) | Price | `hold` | `sold` | `cooldown` | Decisions |
|---|---|---|---|---|---|
| 0 | 1 | -1 | 0 | 0 | Base Cases |
| 1 | 2 | -1 | 1 | 0 | `hold=max(-1, 0-2)=-1`, `sold=-1+2=1`, `cooldown=max(0, 0)=0` |
| 2 | 3 | -1 | 2 | 1 | `hold=max(-1, 0-3)=-1`, `sold=-1+3=2`, `cooldown=max(0, 1)=1` |
| 3 | 0 | 1 | 1 | 2 | `hold=max(-1, 1-0)=1`,  `sold=-1+0=-1` (not possible), `cooldown=max(1, 2)=2` |
| 4 | 2 | 1 | 3 | 2 | `hold=max(1, 2-2)=1`,   `sold=1+2=3`, `cooldown=max(2, 1)=2` |
- **Result**: `3` (Buy at 1, sell at 2, cooldown at 3, buy at 0, sell at 2. Total profit = 1 + 2 = 3).

---

*<- [[DP_on_Stocks_Infinite_Transactions|Infinite Transactions]] · [[DP_on_Stocks_With_Fee|With Transaction Fee ->]]*
