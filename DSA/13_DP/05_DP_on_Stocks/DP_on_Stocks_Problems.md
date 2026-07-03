---
tags: [dsa, dp, stocks, problems, exercises]
links: ["[[05_DP_on_Stocks_Index]]", "[[DP_on_Stocks_K_Transactions]]", "[[../06_DP_on_Trees/06_DP_on_Trees_Index]]"]
---

# DP on Stocks -- Problems & Exercises

*<- [[DP_on_Stocks_K_Transactions|K Transactions]] · [[../06_DP_on_Trees/06_DP_on_Trees_Index|DP on Trees ->]]*

---

## The Complete Stock DP Series Summary

| # | Problem | Constraints | Optimal Time | Optimal Space | Link |
|---|---------|-------------|--------------|---------------|------|
| 1 | Buy & Sell Stock I | 1 transaction | $O(n)$ | $O(1)$ | [LC 121](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) |
| 2 | Buy & Sell Stock II | Infinite transactions | $O(n)$ | $O(1)$ | [LC 122](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/) |
| 3 | Buy & Sell Stock III | Max 2 transactions | $O(n)$ | $O(1)$ | [LC 123](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/) |
| 4 | Buy & Sell Stock IV | Max K transactions | $O(n \cdot k)$ | $O(k)$ | [LC 188](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/) |
| 5 | Buy & Sell Stock with Cooldown | Infinite, 1-day cooldown | $O(n)$ | $O(1)$ | [LC 309](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/) |
| 6 | Buy & Sell Stock with Fee | Infinite, transaction fee | $O(n)$ | $O(1)$ | [LC 714](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/) |

---

## Worked Solution: Unified Stock DP Solver (Standard State Template)

Any stock DP problem can be solved by maintaining states:
- `hold[j]`: max profit at current day holding a stock, after making `j` buys.
- `free[j]`: max profit at current day not holding a stock, after making `j` sells.

For each price `p = prices[i]`:
- `hold[j] = max(hold[j], free[j-1] - p)`
- `free[j] = max(free[j], hold[j] + p - fee)`

This standard structure helps you write any complex combination of transaction counts, fees, or cooldown constraints during interviews without starting from scratch.

---

*<- [[DP_on_Stocks_K_Transactions|K Transactions]] · [[../06_DP_on_Trees/06_DP_on_Trees_Index|DP on Trees ->]]*
