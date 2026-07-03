---
tags: [dsa, dp, stocks, infinite-transactions]
links: ["[[05_DP_on_Stocks_Index]]", "[[DP_on_Stocks_Single_Transaction]]", "[[DP_on_Stocks_With_Cooldown]]"]
---

# DP on Stocks -- Infinite Transactions (LC 122)

*<- [[DP_on_Stocks_Single_Transaction|Single Transaction]] · [[DP_on_Stocks_With_Cooldown|With Cooldown ->]]*

---

## The Problem
You are given an integer array `prices` where `prices[i]` is the price of a given stock on the $i$-th day. On each day, you may decide to buy and/or sell the stock. You can only hold at most one share of the stock at any time. However, you can buy it then immediately sell it on the same day.
- Return the maximum profit you can achieve.

---

## Approach A: Greedy Slope Accumulation
- **Intuition**: If we look at the stock prices graph, any transaction spanning multiple days of upward price movement is equal to the sum of daily profits along that upward slope.
- Therefore, we can greedily capture profit every time the price goes up from one day to the next: if `prices[i] > prices[i-1]`, we buy at `i-1` and sell at `i`.

```cpp
#include <vector>

int maxProfitGreedy(vector<int>& prices) {
    int maxProfit = 0;
    for (int i = 1; i < (int)prices.size(); i++) {
        if (prices[i] > prices[i - 1]) {
            maxProfit += prices[i] - prices[i - 1]; // accumulate positive slopes
        }
    }
    return maxProfit;
}
// Time Complexity: O(n)
// Space Complexity: O(1)
```

---

## Approach B: DP State Machine (Standard Template for Stocks)
To solve more complex variants (like cooldown or transaction fees), the greedy slope method fails. We must use a **State Machine DP**.

At any day `i`, you are in one of two states:
1. **Hold State (`hold`)**: You currently own one share of stock.
   - You either reached this state by holding the stock from the previous day (`hold` stays same).
   - Or you bought the stock today: `free - prices[i]` (where `free` is the max profit of not owning a stock yesterday).
   - **Formula**: `hold = max(hold, free - prices[i])`
2. **Free State (`free`)**: You do not own any stock.
   - You either reached this state by staying free from the previous day (`free` stays same).
   - Or you sold your stock today: `hold + prices[i]`.
   - **Formula**: `free = max(free, hold + prices[i])`

- **Base Cases (Day 0)**:
  - `hold = -prices[0]` (buying on day 0 costs `prices[0]`)
  - `free = 0` (starting profit is 0)

```cpp
#include <vector>
#include <algorithm>

int maxProfitDP(vector<int>& prices) {
    int n = prices.size();
    if (n == 0) return 0;

    int hold = -prices[0];
    int free = 0;

    for (int i = 1; i < n; i++) {
        int nextHold = max(hold, free - prices[i]);
        int nextFree = max(free, hold + prices[i]);
        hold = nextHold;
        free = nextFree;
    }
    return free; // final profit is maximized when we don't hold any stock
}
// Time Complexity: O(n)
// Space Complexity: O(1)
```

### Dry Run Variable State Table:
For `prices = [7, 1, 5, 3, 6, 4]`:
| Iteration (i) | Price | `hold` state | `free` state | Transition Decisions |
|---|---|---|---|---|
| Initial | - | -7 | 0 | Base cases |
| i = 1 | 1 | -1 | 0 | `hold=max(-7, 0-1)=-1`, `free=max(0, -7+1)=0` |
| i = 2 | 5 | -1 | 4 | `hold=max(-1, 0-5)=-1`, `free=max(0, -1+5)=4` |
| i = 3 | 3 | 1 | 4 | `hold=max(-1, 4-3)=1`,  `free=max(4, -1+3)=4` |
| i = 4 | 6 | 1 | 7 | `hold=max(1, 4-6)=1`,   `free=max(4, 1+6)=7` |
| i = 5 | 4 | 3 | 7 | `hold=max(1, 7-4)=3`,   `free=max(7, 1+4)=7` |
- **Result**: `7` (Correct).

---

*<- [[DP_on_Stocks_Single_Transaction|Single Transaction]] · [[DP_on_Stocks_With_Cooldown|With Cooldown ->]]*
