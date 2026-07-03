---
tags: [dsa, tries, tricky, hard, palindrome-pairs]
links: ["[[08_Tries_Index]]", "[[08_Tries_Problems_and_Exercises]]", "[[../09_Heap_Priority_Queue/00_Index]]"]
---

# Tries -- Tricky & Higher-Order

*<- [[08_Tries_Problems_and_Exercises\|Problems]] · [[../09_Heap_Priority_Queue/00_Index\|Heap ->]]*

---

## 1. Palindrome Pairs (LC 336)

**Why tricky**: Given a list of unique words, find all pairs `(i, j)` where `words[i] + words[j]` is a palindrome. O(n²L) naive is too slow. 
By utilizing a hash map or Trie to store the words and their reversed forms, we can search for valid matching prefixes/suffixes in $O(n \cdot L^2)$ time.

### Three cases for `words[i] + words[j]` to be a palindrome:

#### Case 1: `len(words[i]) == len(words[j])`
`words[i] = "abcd"` and `words[j] = "dcba"`. 
- `words[i] + words[j] = "abcddcba"` (Palindrome).
- **Condition**: `words[j]` is the exact reverse of `words[i]`.

#### Case 2: `len(words[i]) > len(words[j])`
`words[i] = "lls"` and `words[j] = "sss"` is not a valid match, but if `words[i] = "sssll"` and `words[j] = "sss"`, we split `words[i]` into a prefix and a suffix.
Let `words[i]` be split into `prefix = "sss"` and `suffix = "ll"`.
- If `suffix = "ll"` is a palindrome, we look for the reverse of `prefix = "sss"`, which is `"sss"`.
- Since `"sss"` exists in the dictionary, `words[i] + words[j] = "sssll" + "sss" = "sssllsss"` (not a palindrome). 
- Wait, the correct order is: `words[i] + reverse(prefix)` where `suffix` is a palindrome.
- If `words[i] = "llsss"` and we split into `prefix = "ll"` (palindrome) and `suffix = "sss"`. Then `words[j] = "sss"`.
- `words[j] + words[i] = "sss" + "llsss" = "sssllsss"` (Palindrome).
- **Condition**: Split `words[i]` into `[prefix, suffix]`. If `suffix` is a palindrome, we search for `reverse(prefix)` to append to the right: `words[i] + reverse(prefix)`.

#### Case 3: `len(words[i]) < len(words[j])`
This is symmetric to Case 2. If we process `words[j]` by splitting it, it resolves to Case 2 from the other word's perspective. Thus, by splitting every word at every possible index `cut` from `0` to `L`, we cover all combinations.

### Step-by-Step Split Tracing table for `w = "abcd"`
| Cut | Prefix | Suffix | Prefix Palindrome? | Suffix Palindrome? | Action if Yes |
|---|---|---|---|---|---|
| 0 | `""` | `"abcd"` | Yes (empty) | No | If prefix is palindrome, look for `reverse(suffix) = "dcba"` to prepend. |
| 1 | `"a"` | `"bcd"` | Yes (`"a"`) | No | Look for `reverse(suffix) = "dcb"` to prepend. |
| 2 | `"ab"` | `"cd"` | No | No | - |
| 3 | `"abc"` | `"d"` | No | Yes (`"d"`) | If suffix is palindrome, look for `reverse(prefix) = "cba"` to append. |
| 4 | `"abcd"` | `""` | No | Yes (empty) | If suffix is palindrome, look for `reverse(prefix) = "dcba"` to append. |

```cpp
bool isPalin(const string& s, int lo, int hi) {
    while (lo < hi) { if (s[lo++] != s[hi--]) return false; }
    return true;
}

vector<vector<int>> palindromePairs(vector<string>& words) {
    unordered_map<string, int> wordMap;
    for (int i = 0; i < (int)words.size(); i++) wordMap[words[i]] = i;

    vector<vector<int>> result;

    for (int i = 0; i < (int)words.size(); i++) {
        const string& w = words[i];
        int n = w.size();

        for (int cut = 0; cut <= n; cut++) {
            // Case 1 & 2: Split into prefix w[0..cut-1] and suffix w[cut..n-1]
            // If suffix is a palindrome, we search for reverse(prefix) to append on the right
            if (isPalin(w, cut, n - 1)) {
                string revPrefix = string(w.begin(), w.begin() + cut);
                reverse(revPrefix.begin(), revPrefix.end());
                auto it = wordMap.find(revPrefix);
                if (it != wordMap.end() && it->second != i) {
                    result.push_back({i, it->second}); // words[i] + words[it->second]
                }
            }
            
            // Case 3: If prefix is a palindrome, we search for reverse(suffix) to prepend on the left
            // cut > 0 avoids duplicate checks for empty prefix/suffix at boundaries
            if (cut > 0 && isPalin(w, 0, cut - 1)) {
                string revSuffix = string(w.begin() + cut, w.end());
                reverse(revSuffix.begin(), revSuffix.end());
                auto it = wordMap.find(revSuffix);
                if (it != wordMap.end() && it->second != i) {
                    result.push_back({it->second, i}); // words[it->second] + words[i]
                }
            }
        }
    }
    return result;
}
// Time Complexity: O(n * L²) — where n is the number of words, L is max word length.
//                  For each word, we check L cuts. For each cut, we run isPalin of length L.
// Space Complexity: O(n * L) to store the word map.
```

---

## 2. Map Sum Pairs (LC 677)

**Why tricky**: Query sum of all values whose keys start with a given prefix. 
- If we do this naively on query, it takes $O(n \cdot L)$ time where we scan all $n$ keys.
- By utilizing a Trie where each node stores a cumulative `sum` of all values in its subtree, we query the sum in optimal $O(L)$ time.
- **Tricky Edge Case**: Re-inserting an existing key with a new value. We must calculate the difference (`delta = new_val - old_val`) and propagate this delta down the tree during insertion.

### Delta Propagation Trace Table:
1. `insert("apple", 3)`:
   - key `"apple"` doesn't exist (old_val = 0). `delta = 3 - 0 = 3`.
   - Nodes `a -> p -> p -> l -> e` all have their `sum` incremented by `3`.
2. `insert("app", 2)`:
   - key `"app"` doesn't exist (old_val = 0). `delta = 2 - 0 = 2`.
   - Nodes `a -> p -> p` have their `sum` incremented by `2`.
   - Subtree sums now: `a.sum = 5`, `p.sum = 5`, `p.sum = 5`, `l.sum = 3`, `e.sum = 3`.
3. `insert("apple", 5)` (re-insert):
   - key `"apple"` exists with val `3`. `delta = 5 - 3 = 2`.
   - Nodes `a -> p -> p -> l -> e` have their `sum` incremented by `2`.
   - Subtree sums now: `a.sum = 7`, `p.sum = 7`, `p.sum = 7`, `l.sum = 5`, `e.sum = 5`.

```cpp
class MapSum {
    struct MSNode {
        MSNode* children[26];
        int val;       // stored value at this word node (0 if not end of word)
        int sum;       // sum of all values in the subtree rooted here
        MSNode() : val(0), sum(0) {
            for (int i = 0; i < 26; i++) children[i] = nullptr;
        }
    };
    MSNode* root;
    unordered_map<string, int> keyMap; // track existing key-value pairs

public:
    MapSum() { root = new MSNode(); }

    void insert(string key, int val) {
        int delta = val - keyMap[key]; // calculate change delta (handles overwriting)
        keyMap[key] = val;
        MSNode* cur = root;
        for (char c : key) {
            int i = c - 'a';
            if (!cur->children[i]) cur->children[i] = new MSNode();
            cur = cur->children[i];
            cur->sum += delta; // propagate delta through the insertion path
        }
        cur->val = val;
    }

    int sum(string prefix) {
        MSNode* cur = root;
        for (char c : prefix) {
            int i = c - 'a';
            if (!cur->children[i]) return 0;
            cur = cur->children[i];
        }
        return cur->sum; // O(L) retrieval of the precalculated subtree sum
    }
};
```

---

## 3. Maximum XOR of Two Numbers with Queries (LC 1707)

**Why tricky**: Given nums[] and queries[xi, mi], for each query find max XOR of xi with any element in nums where element <= mi. Offline: sort queries and nums by mi, add elements to XOR trie incrementally.

```cpp
int maximumXorWithQueries(vector<int>& nums, vector<vector<int>>& queries) {
    // Sort queries offline by mi (threshold)
    int Q = queries.size();
    vector<int> qIdx(Q);
    iota(qIdx.begin(), qIdx.end(), 0);  // C++17: fill 0,1,2,...,Q-1
    sort(qIdx.begin(), qIdx.end(), [&](int a, int b) {
        return queries[a][1] < queries[b][1];
    });
    sort(nums.begin(), nums.end());

    XorTrie trie;  // from Pattern 5 in 01_Patterns.md
    vector<int> answers(Q, -1);
    int ni = 0;

    for (int qi : qIdx) {
        int x = queries[qi][0], m = queries[qi][1];
        // Add all nums <= m to trie
        while (ni < (int)nums.size() && nums[ni] <= m) {
            trie.insert(nums[ni++]);
        }
        if (ni > 0) answers[qi] = trie.maxXOR(x);
    }
    return *max_element(answers.begin(), answers.end());
}
// Time: O((n + Q) log(max_value)), Space: O(n * 32)
// iota: fills range with sequentially increasing values -- C++11
```

---

## 4. Count Distinct Substrings Using Suffix Trie

**Why tricky**: Every substring is a prefix of some suffix. Insert all suffixes into a trie. Count of distinct substrings = total number of nodes in the trie (each node = one unique character extending a unique prefix).

```cpp
int countDistinctSubstrings(const string& s) {
    // Insert all suffixes
    Trie trie;
    int distinctCount = 0;

    // Instead of counting nodes, count insertions of NEW nodes
    // Modified insert that returns new nodes added
    struct CountingTrie {
        TrieNode* root;
        CountingTrie() { root = new TrieNode(); }

        int insertAndCount(const string& s, int start) {
            TrieNode* cur = root;
            int newNodes = 0;
            for (int i = start; i < (int)s.size(); i++) {
                int idx = s[i] - 'a';
                if (!cur->children[idx]) {
                    cur->children[idx] = new TrieNode();
                    newNodes++;
                }
                cur = cur->children[idx];
            }
            cur->isEnd = true;
            return newNodes;
        }
    } ctrie;

    for (int i = 0; i < (int)s.size(); i++) {
        distinctCount += ctrie.insertAndCount(s, i);
    }
    return distinctCount;
}
// Time: O(n²) -- n suffixes, average length n/2 each
// For production use: Suffix Array achieves O(n log n)
```

---

## 5. Compressed Trie / Radix Tree (Concept)

**Why tricky**: Standard trie has one character per node. Compressed trie merges chains of single-child nodes into one edge labeled with a string. Reduces space from O(total chars) to O(words).

```
Standard:  c -> a -> t (*)
           c -> a -> r -> (*) -> t (*)

Compressed: "ca" -> "t" (*)
                 -> "r" (*) -> "t" (*)
```

```
Used in:
- IP routing tables (Patricia trie / CIDR prefix matching)
- Git object store (pack file index)
- DNS lookup optimization

Interview relevance: know the concept, explain space improvement.
O(words) nodes instead of O(total_characters) nodes.
Each edge stores a string range [start, length] into a string pool.
```

---

## Problem List

| # | Problem | Difficulty | Link |
|---|---------|------------|------|
| 1 | Palindrome Pairs | Hard | [LC 336](https://leetcode.com/problems/palindrome-pairs/) |
| 2 | Map Sum Pairs | Medium | [LC 677](https://leetcode.com/problems/map-sum-pairs/) |
| 3 | Maximum XOR with Element from Array | Hard | [LC 1707](https://leetcode.com/problems/maximum-xor-with-an-element-from-array/) |
| 4 | Word Squares | Hard | [LC 425](https://leetcode.com/problems/word-squares/) |
| 5 | Concatenated Words | Hard | [LC 472](https://leetcode.com/problems/concatenated-words/) |
| 6 | Stream of Characters | Hard | [LC 1032](https://leetcode.com/problems/stream-of-characters/) |

---

*<- [[08_Tries_Problems_and_Exercises\|Problems]] · [[../09_Heap_Priority_Queue/00_Index\|Heap ->]]*
