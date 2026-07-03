---
tags: [dsa, hashing, unordered-map, unordered-set, rolling-hash]
links: ["[[00_Index]]", "[[02_Prefix_Sums]]", "[[04_Key_Algorithms]]"]
---

# Hashing, Maps & Sets

*← [[02_Prefix_Sums|Prefix Sums]] · [[04_Key_Algorithms|Key Algorithms →]]*

---

## Why Hashing?

You often need to answer: "Have I seen this value before?" or "How many times did this appear?"

**Array/brute force**: scan every previous element → O(n) per lookup.  
**Hash table**: O(1) average per lookup. Magic.

A hash function converts a key (integer, string, etc.) into an array index. Good hash functions spread keys uniformly to minimize collisions.

---

## Pattern 1: Frequency Counting

**Use when**: You need to count how many times each element appears.

```cpp
#include <unordered_map>
#include <vector>

// Count frequency of each element
std::unordered_map<int, int> freq;
std::vector<int> nums = {1, 2, 2, 3, 3, 3};

for (int x : nums) {
    freq[x]++;       // if key not present, it's default-constructed to 0, then incremented
}

// Check if element exists
if (freq.count(3)) {  // count() returns 0 or 1 for unordered_map
    // 3 exists
}
// Or:
if (freq.find(3) != freq.end()) {
    // 3 exists
}
// Or (C++20):
if (freq.contains(3)) {  // .contains() — cleaner, C++20
    // 3 exists
}

// Iterate
for (auto& [key, val] : freq) {  // C++17 structured binding
    std::cout << key << " appears " << val << " times\n";
}
```

---

## Pattern 2: Existence Check (unordered_set)

**Use when**: You only need to know IF something exists (not how many times).

```cpp
#include <unordered_set>
#include <vector>

std::vector<int> nums = {1, 2, 3, 4, 5};
std::unordered_set<int> seen(nums.begin(), nums.end()); // O(n) build

bool hasFour = seen.count(4);        // O(1) average
seen.insert(6);                       // O(1) average
seen.erase(2);                        // O(1) average

// Check two arrays for common elements: O(n + m) total
std::vector<int> a = {1,2,3}, b = {3,4,5};
std::unordered_set<int> setA(a.begin(), a.end());
for (int x : b) {
    if (setA.count(x)) {
        // x is in both arrays
    }
}
```

---

## Pattern 3: Two-Sum Pattern (Value → Index Mapping)

**Core idea**: Store `value → index` in a map. For each new element, check if its complement was already seen.

```cpp
#include <unordered_map>
#include <vector>

// LeetCode 1 — Two Sum
std::vector<int> twoSum(std::vector<int>& nums, int target) {
    std::unordered_map<int, int> indexMap;  // value → index
    
    for (int i = 0; i < (int)nums.size(); i++) {
        int complement = target - nums[i];
        
        if (indexMap.count(complement)) {  // complement seen before?
            return {indexMap[complement], i};
        }
        indexMap[nums[i]] = i;             // store this element for future lookups
    }
    return {};  // no solution (problem guarantees one exists)
}
// Time: O(n), Space: O(n)
```

---

## Pattern 4: Grouping by Hash (Anagram Grouping)

**When**: Group elements by some property that can be reduced to a hashable key.

```cpp
#include <unordered_map>
#include <vector>
#include <string>
#include <algorithm>

// LeetCode 49 — Group Anagrams
// Key insight: anagrams, when sorted, are identical. Use sorted string as key.
std::vector<std::vector<std::string>> groupAnagrams(
    std::vector<std::string>& strs
) {
    std::unordered_map<std::string, std::vector<std::string>> groups;
    
    for (std::string& s : strs) {
        std::string key = s;
        std::sort(key.begin(), key.end());  // sort to get canonical form
        groups[key].push_back(s);
    }
    
    std::vector<std::vector<std::string>> result;
    for (auto& [key, group] : groups) {
        result.push_back(group);
    }
    return result;
}
// Time: O(n * k log k) where k = max string length
// Space: O(n * k)
```

---

## Pattern 5: Custom Hash (Critical for Codeforces / Anti-Hack)

**Why**: The default `std::hash<int>` in `unordered_map` is deterministic and can be reverse-engineered by adversaries on platforms like Codeforces to create O(n) hash collisions.

```cpp
#include <unordered_map>

// Custom hash to avoid adversarial collision attacks
struct CustomHash {
    static uint64_t splitmix64(uint64_t x) {
        // splitmix64 scrambles the bits — extremely fast and collision-resistant
        x += 0x9e3779b97f4a7c15;
        x = (x ^ (x >> 30)) * 0xbf58476d1ce4e5b9;
        x = (x ^ (x >> 27)) * 0x94d049bb133111eb;
        return x ^ (x >> 31);
    }
    
    size_t operator()(uint64_t x) const {
        // Seed with random value to prevent hack reproducibility
        static const uint64_t SEED = std::chrono::steady_clock::now()
                                       .time_since_epoch().count();
        return splitmix64(x + SEED);
    }
};

// Usage: drop-in replacement
std::unordered_map<int, int, CustomHash> safe_map;
std::unordered_set<int, CustomHash> safe_set;
// Use this on Codeforces whenever you use unordered containers
```

---

## Pattern 6: Rolling Hash (Rabin-Karp)

**Why**: Sometimes you need to check if two substrings are equal in O(1) time (e.g., find all occurrences of a pattern in text, find duplicate substrings). Hashing the entire substring character by character is O(k). **Rolling hash** computes the next window's hash from the previous one in O(1) by using polynomial hashing.

### Concept

Treat the string as a number in some base `B` modulo a large prime `MOD`.

```
hash("abc") = 'a'*B² + 'b'*B¹ + 'c'*B⁰  (mod MOD)

Sliding window: remove leftmost, add rightmost:
old = "abc" → hash = a*B² + b*B + c
new = "bcd" → hash = b*B² + c*B + d
                   = (old - a*B²) * B + d
```

```cpp
#include <string>
#include <vector>

// Find all starting indices where pattern appears in text
// Rabin-Karp Algorithm
std::vector<int> rabinKarp(const std::string& text, const std::string& pattern) {
    const long long MOD = 1e9 + 7;  // large prime
    const long long BASE = 31;       // prime larger than alphabet size

    int n = text.size(), m = pattern.size();
    std::vector<int> result;

    if (m > n) return result;

    // Precompute BASE^(m-1) mod MOD — needed to remove the leading character
    long long highPow = 1;
    for (int i = 0; i < m - 1; i++) {
        highPow = (highPow * BASE) % MOD;
    }

    // Compute hash of pattern and first window of text
    long long patHash = 0, winHash = 0;
    for (int i = 0; i < m; i++) {
        patHash = (patHash * BASE + (pattern[i] - 'a' + 1)) % MOD;
        winHash = (winHash * BASE + (text[i]   - 'a' + 1)) % MOD;
    }

    // Slide window across text
    for (int i = 0; i <= n - m; i++) {
        if (winHash == patHash) {
            // Hashes match — verify character by character (avoid false positives)
            if (text.substr(i, m) == pattern) {
                result.push_back(i);
            }
        }
        // Roll the hash to next window
        if (i < n - m) {
            winHash = (winHash - (long long)(text[i] - 'a' + 1) * highPow % MOD + MOD) % MOD;
            winHash = (winHash * BASE + (text[i + m] - 'a' + 1)) % MOD;
        }
    }
    return result;
}
// Time: O(n + m) average, O(n*m) worst case (many hash collisions)
// Use double hashing (two different MOD/BASE pairs) to reduce false positives to near zero
```

### Rolling Hash for Duplicate Substrings (Binary Search + Rolling Hash)

```cpp
// LC 1044 — Longest Duplicate Substring
// Approach: Binary search on length L, then use rolling hash to check
//           if any substring of length L appears twice
#include <string>
#include <unordered_set>

bool hasDuplicate(const std::string& s, int L) {
    const long long MOD = (1LL << 61) - 1;  // Mersenne prime — fewer collisions
    const long long BASE = 131;

    auto mul = [&](long long a, long long b) -> long long {
        // Overflow-safe multiplication mod Mersenne prime
        __uint128_t res = (__uint128_t)a * b;
        // __uint128_t is GCC extension for 128-bit int — C++ standard doesn't have it
        // but it's supported in g++ (the compiler used in competitive programming)
        return (long long)((res >> 61) + (res & MOD)) % MOD;
    };

    long long hashVal = 0, highPow = 1;
    for (int i = 0; i < L; i++) {
        hashVal = (mul(hashVal, BASE) + s[i]) % MOD;
        if (i < L - 1) highPow = mul(highPow, BASE);
    }

    std::unordered_set<long long> seen;
    seen.insert(hashVal);

    for (int i = L; i < (int)s.size(); i++) {
        hashVal = (mul(hashVal - mul(highPow, s[i - L]) + MOD, BASE) + s[i]) % MOD;
        if (seen.count(hashVal)) return true;
        seen.insert(hashVal);
    }
    return false;
}
```

---

## Pattern 7: Coordinate Compression

**When**: Values are huge (up to 10^9) but you only care about relative order. Map them to small indices so you can use arrays instead of maps.

```cpp
#include <vector>
#include <algorithm>

std::vector<int> compress(std::vector<int> arr) {
    std::vector<int> sorted = arr;
    std::sort(sorted.begin(), sorted.end());
    sorted.erase(std::unique(sorted.begin(), sorted.end()), sorted.end()); // remove dups

    for (int& x : arr) {
        // Replace each value with its rank (0-indexed)
        x = (int)(std::lower_bound(sorted.begin(), sorted.end(), x) - sorted.begin());
    }
    return arr;
}
// Example: [100, 500, 200, 100] → [0, 2, 1, 0]
// Now you can use arr values as array indices safely
```

---

## Examples

### Example 1 — LC 1: Two Sum (Easy)
Already shown in Pattern 3.

### Example 2 — LC 49: Group Anagrams (Medium)
Already shown in Pattern 4.

### Example 3 — LC 128: Longest Consecutive Sequence (Medium)

```cpp
// Key insight: only start counting from the beginning of a sequence
// (i.e., when x-1 is NOT in the set)
int longestConsecutive(std::vector<int>& nums) {
    std::unordered_set<int> s(nums.begin(), nums.end());
    int best = 0;
    for (int x : s) {
        if (!s.count(x - 1)) {   // x is the start of a sequence
            int len = 1;
            while (s.count(x + len)) len++;
            best = std::max(best, len);
        }
    }
    return best;
}
// Time: O(n) — each element is visited at most twice
```

### Example 4 — LC 217: Contains Duplicate (Easy)

```cpp
bool containsDuplicate(std::vector<int>& nums) {
    std::unordered_set<int> seen;
    for (int x : nums) {
        if (seen.count(x)) return true;
        seen.insert(x);
    }
    return false;
}
```

---

## Exercises

| # | Problem | Platform | Difficulty | Pattern |
|---|---------|----------|------------|---------|
| 1 | [1. Two Sum](https://leetcode.com/problems/two-sum/) | LeetCode | 🟢 Easy | HashMap |
| 2 | [217. Contains Duplicate](https://leetcode.com/problems/contains-duplicate/) | LeetCode | 🟢 Easy | HashSet |
| 3 | [242. Valid Anagram](https://leetcode.com/problems/valid-anagram/) | LeetCode | 🟢 Easy | Frequency count |
| 4 | [49. Group Anagrams](https://leetcode.com/problems/group-anagrams/) | LeetCode | 🟡 Medium | Group by hash |
| 5 | [347. Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) | LeetCode | 🟡 Medium | Freq count + sort/heap |
| 6 | [128. Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/) | LeetCode | 🟡 Medium | HashSet + start-of-seq |
| 7 | [454. 4Sum II](https://leetcode.com/problems/4sum-ii/) | LeetCode | 🟡 Medium | HashMap (split pairs) |
| 8 | [1044. Longest Duplicate Substring](https://leetcode.com/problems/longest-duplicate-substring/) | LeetCode | 🔴 Hard | Rolling Hash + Binary Search |
| 9 | [187. Repeated DNA Sequences](https://leetcode.com/problems/repeated-dna-sequences/) | LeetCode | 🟡 Medium | Rolling Hash / HashSet |
| 10 | [Rabin-Karp on GFG](https://www.geeksforgeeks.org/rabin-karp-algorithm-for-pattern-searching/) | GFG | 🟡 Medium | Rabin-Karp |

---

*← [[02_Prefix_Sums|Prefix Sums]] · [[04_Key_Algorithms|Key Algorithms →]]*
