---
tags: [dsa, greedy, patterns, knapsack]
links: ["[[14_Greedy_Index]]", "[[14_Greedy_Problems_and_Exercises]]"]
---

# Greedy Algorithms -- Patterns

*<- [[14_Greedy_Index|Index]] · [[14_Greedy_Problems_and_Exercises|Problems ->]]*

---

## What is a Greedy Algorithm?

A greedy algorithm makes the **locally optimal choice** at each stage with the hope of finding a global optimum. 

### Core Properties:
1. **Greedy Choice Property**: A global optimum can be reached by making local optimal choices.
2. **Optimal Substructure**: The optimal solution to the problem contains optimal solutions to subproblems.

Unlike Dynamic Programming, which evaluates all possible outcomes at each step, Greedy **locks in a decision** immediately and never backtracks. This makes it faster ($O(n)$ or $O(n \log n)$) but requires rigorous proof of correctness.

---

## Pattern 1: Value/Weight Density (Fractional Knapsack)

**The Problem**: Given weights and values of $n$ items, put these items in a knapsack of capacity $W$. You can break items (take fractions) for maximum total value.

### Greedy Choice
Sort items by their **value-to-weight density** ($v_i / w_i$) in descending order. Take as much of the densest item as possible, then move to the next densest.

```cpp
#include <vector>
#include <algorithm>

struct Item {
    int value;
    int weight;
};

// Custom density sorting comparator
bool cmp(const Item& a, const Item& b) {
    double r1 = (double)a.value / a.weight;
    double r2 = (double)b.value / b.weight;
    return r1 > r2; // sort descending
}

double fractionalKnapsack(int W, vector<Item>& items) {
    sort(items.begin(), items.end(), cmp); // Step 1: Sort by density

    double totalValue = 0.0;
    int currentWeight = 0;

    for (const auto& item : items) {
        if (currentWeight + item.weight <= W) {
            // Take the whole item
            currentWeight += item.weight;
            totalValue += item.value;
        } else {
            // Take a fraction of the remaining item
            int remaining = W - currentWeight;
            totalValue += item.value * ((double)remaining / item.weight);
            break; // knapsack is full
        }
    }
    return totalValue;
}
// Time Complexity: O(n log n) due to sorting
// Space Complexity: O(1) (excluding input)
```

---

## Pattern 2: Max Reachability Tracking (Jump Game -- LC 55)

**The Problem**: You are given an integer array `nums` representing your maximum jump length at each position. Start at index 0. Determine if you can reach the last index.

### Greedy Choice
Instead of checking all possible jump landing positions (which is $O(n^2)$ DP), maintain the **maximum reachable index seen so far** (`maxReach`). If at any point the current index `i > maxReach`, you cannot proceed further (return `false`).

```cpp
#include <vector>
#include <algorithm>

bool canJump(vector<int>& nums) {
    int maxReach = 0;
    int n = nums.size();

    for (int i = 0; i < n; i++) {
        if (i > maxReach) return false; // current index is unreachable
        
        maxReach = max(maxReach, i + nums[i]);
        if (maxReach >= n - 1) return true; // optimized exit
    }
    return true;
}
// Time Complexity: O(n) — single pass
// Space Complexity: O(1)
```

### Dry Run Variable State Table:
For `nums = [2, 3, 1, 1, 4]`:
| Index (i) | Num | `i > maxReach`? | `i + nums[i]` (New Potential Reach) | `maxReach` |
|---|---|---|---|---|
| Initial | - | - | - | 0 |
| i = 0 | 2 | No | 0 + 2 = 2 | 2 |
| i = 1 | 3 | No | 1 + 3 = 4 | 4 |
| i = 2 | 1 | No | 2 + 1 = 3 | 4 |
| i = 3 | 1 | No | 3 + 1 = 4 | 4 |
| i = 4 | 4 | No | 4 + 4 = 8 | 8 |
- **Result**: `true` (`maxReach >= 4`).

---

## Pattern 3: Circular Balance Reset (Gas Station -- LC 134)

**The Problem**: There are $n$ gas stations along a circular route, where the amount of gas at the $i$-th station is `gas[i]`. You have a car with an unlimited gas tank and it costs `cost[i]` of gas to travel from the $i$-th station to its next $(i+1)$-th station. Return the starting gas station's index if you can travel around the circuit once, otherwise return -1.

### Greedy Choice
- If the total gas is less than the total cost, a solution is mathematically impossible (return `-1`).
- Otherwise, a unique starting point **must exist**. 
- Start from index 0. Track current tank balance. If at any station `i`, `currentTank < 0`, it means we cannot travel from our current starting point to `i`. Since any station *between* our start and `i` would also have run out of gas, we can greedily **reset our starting point to `i + 1`** and reset `currentTank` to 0.

```cpp
#include <vector>
#include <numeric>

int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
    int totalGas = accumulate(gas.begin(), gas.end(), 0);
    int totalCost = accumulate(cost.begin(), cost.end(), 0);

    if (totalGas < totalCost) return -1; // impossible to complete circular trip

    int start = 0;
    int currentTank = 0;

    for (int i = 0; i < (int)gas.size(); i++) {
        currentTank += gas[i] - cost[i];
        if (currentTank < 0) {
            start = i + 1; // start index candidate reset
            currentTank = 0; // reset tank balance
        }
    }
    return start;
}
// Time Complexity: O(n)
// Space Complexity: O(1)
```

---

## Pattern 4: Frequency Ordering (Hand of Straights -- LC 846)

**The Problem**: Alice has some cards. Group them into sets of `groupSize` such that each group contains `groupSize` consecutive cards.

### Greedy Choice
To form consecutive groups, we must start with the **smallest card available**.
- Use a sorted frequency map (in C++, `std::map` is ordered by keys).
- Find the smallest card `start` with count > 0.
- Check if we have consecutive cards `start, start + 1, ..., start + groupSize - 1` available. If not, return `false`.
- Decrement their counts. Repeat.

```cpp
#include <vector>
#include <map>

bool isNStraightHand(vector<int>& hand, int groupSize) {
    if (hand.size() % groupSize != 0) return false;

    // Ordered map to store counts sorted by card value
    map<int, int> countMap;
    for (int card : hand) countMap[card]++;

    for (auto& [card, count] : countMap) {
        if (count > 0) {
            int currentCount = count;
            // We need currentCount groups starting with 'card'
            for (int i = 0; i < groupSize; i++) {
                int nextCard = card + i;
                if (countMap[nextCard] < currentCount) {
                    return false; // consecutive card not available in sufficient quantity
                }
                countMap[nextCard] -= currentCount; // consume cards
            }
        }
    }
    return true;
}
// Time Complexity: O(n log n) for map insertion + O(n log n) checks
// Space Complexity: O(n) for the map
```

---

*<- [[14_Greedy_Index|Index]] · [[14_Greedy_Problems_and_Exercises|Problems ->]]*
