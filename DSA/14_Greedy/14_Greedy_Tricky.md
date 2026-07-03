---
tags: [dsa, greedy, tricky, hard, candy, huffman-coding]
links: ["[[14_Greedy_Index]]", "[[14_Greedy_Problems_and_Exercises]]", "[[../15_Intervals/15_Intervals_Index]]"]
---

# Greedy Algorithms -- Tricky & Higher-Order

*<- [[14_Greedy_Problems_and_Exercises|Problems]] · [[../15_Intervals/15_Intervals_Index|Intervals ->]]*

---

## 1. Candy (LC 135)

**Why Tricky**: There are $n$ children standing in a line. Each child has a rating value. You must give at least 1 candy to each child, and children with a higher rating than their neighbors must get more candies than their neighbors. Find the minimum candies needed.
- **Why one pass fails**: If you only look at the left neighbor, you might violate the right neighbor constraint when ratings drop.

### The Solution: Two-Pass Greedy
1. **Left-to-Right Pass**: Ensure children with higher ratings than their left neighbor get more candies than them.
   `if (ratings[i] > ratings[i-1]) candies[i] = candies[i-1] + 1`
2. **Right-to-Left Pass**: Ensure children with higher ratings than their right neighbor get more candies than them.
   `if (ratings[i] > ratings[i+1]) candies[i] = max(candies[i], candies[i+1] + 1)`

```cpp
#include <vector>
#include <algorithm>
#include <numeric>

int candy(vector<int>& ratings) {
    int n = ratings.size();
    vector<int> candies(n, 1); // give 1 candy to each child initially

    // Pass 1: Left to Right
    for (int i = 1; i < n; i++) {
        if (ratings[i] > ratings[i - 1]) {
            candies[i] = candies[i - 1] + 1;
        }
    }

    // Pass 2: Right to Left
    for (int i = n - 2; i >= 0; i--) {
        if (ratings[i] > ratings[i + 1]) {
            candies[i] = max(candies[i], candies[i + 1] + 1);
        }
    }

    // Sum total candies
    return accumulate(candies.begin(), candies.end(), 0);
}
// Time Complexity: O(n) — two passes
// Space Complexity: O(n) for candies array
```

### Dry Run Variable State Table:
For `ratings = [1, 2, 87, 87, 87, 2, 1]`:
- Initial: `[1, 1, 1, 1, 1, 1, 1]`

| Pass Direction | Iterations | `candies` Array State |
|---|---|---|
| Left-to-Right | i=1: `2 > 1` -> `candies[1] = 2`<br>i=2: `87 > 2` -> `candies[2] = 3`<br>i=3..6: no increases | `[1, 2, 3, 1, 1, 1, 1]` |
| Right-to-Left | i=5: `2 > 1` -> `candies[5] = max(1, 2) = 2`<br>i=4: `87 > 2` -> `candies[4] = max(1, 3) = 3`<br>i=3..0: no increases | `[1, 2, 3, 1, 3, 2, 1]` |
- **Result**: `1 + 2 + 3 + 1 + 3 + 2 + 1 = 13` (Correct).

---

## 2. Partition Labels (LC 763)

**Why Tricky**: We want to partition a string into as many parts as possible so that each letter appears in at most one part.
- **The Solution**:
  1. Record the **last occurrence index** of each character in the string.
  2. Maintain a running partition window: `start = 0`, `end = 0`.
  3. For each index `i` (char `c`), update `end = max(end, last_occurrence[c])`.
  4. If `i == end`, it means all characters seen so far have their last occurrences inside the current partition window. Lock the partition, record its size `end - start + 1`, and update `start = i + 1`.

```cpp
#include <vector>
#include <string>
#include <algorithm>

vector<int> partitionLabels(string s) {
    // Record last occurrences of each character
    int last[26];
    for (int i = 0; i < (int)s.size(); i++) {
        last[s[i] - 'a'] = i;
    }

    vector<int> result;
    int start = 0, end = 0;

    for (int i = 0; i < (int)s.size(); i++) {
        end = max(end, last[s[i] - 'a']); // extend partition window boundary
        if (i == end) {
            result.push_back(end - start + 1); // lock partition
            start = i + 1; // start next partition
        }
    }
    return result;
}
// Time Complexity: O(n)
// Space Complexity: O(1) (size of alphabet 26 is constant)
```

---

## 3. Huffman Coding (Greedy Prefix Codes)

**Why Tricky**: Compress data by assigning variable-length binary codes to characters based on their frequencies.
- **Greedy choice**: Combine the two least frequent characters/nodes at each step into a parent node whose frequency is the sum of the children. This creates a binary tree bottom-up.
- **Huffman Algorithm**:
  1. Put all leaf nodes in a Min-Priority Queue sorted by frequency.
  2. Pop two nodes with smallest frequencies.
  3. Create a parent node with sum frequency, point its children to the popped nodes, and push parent back.
  4. Repeat until only 1 root node remains.
  5. Traverse tree to assign binary codes (0 for left, 1 for right).
- **Complexity**: $O(n \log n)$ where $n$ is unique characters.

```
Frequencies: A:5, B:9, C:12, D:13, E:16, F:45
Merge A(5) & B(9) -> Parent (14). Heap: [C:12, D:13, P1:14, E:16, F:45]
Merge C(12) & D(13) -> Parent (25). Heap: [P1:14, E:16, P2:25, F:45]
...
```

---

*<- [[14_Greedy_Problems_and_Exercises|Problems]] · [[../15_Intervals/15_Intervals_Index|Intervals ->]]*
