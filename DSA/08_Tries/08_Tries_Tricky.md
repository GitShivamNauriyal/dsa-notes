---
tags: [dsa, tries, tricky, hard, palindrome-pairs]
links: ["[[08_Tries_Index]]", "[[08_Tries_Problems_and_Exercises]]", "[[../09_Heap_Priority_Queue/00_Index]]"]
---

# Tries -- Tricky & Higher-Order

*<- [[08_Tries_Problems_and_Exercises|Problems]] · [[../09_Heap_Priority_Queue/00_Index|Heap ->]]*

---

## 1. Palindrome Pairs (LC 336)

**Why tricky**: Given a list of unique words, find all pairs `(i, j)` where `words[i] + words[j]` is a palindrome. O(n²L) naive is too slow. Trie approach: for each word, search for its reverse in the trie while checking palindrome conditions on remaining suffixes.

**Three cases for `words[i] + words[j]` to be a palindrome**:
1. `len(words[i]) == len(words[j])` and `words[j] == reverse(words[i])`
2. `len(words[i]) > len(words[j])`: prefix of words[i] == reverse(words[j]), AND the remaining suffix of words[i] is itself a palindrome
3. `len(words[i]) < len(words[j])`: words[i] == reverse of a suffix of words[j], AND the prefix of words[j] is itself a palindrome

```cpp
bool isPalin(const std::string& s, int lo, int hi) {
    while (lo < hi) { if (s[lo++] != s[hi--]) return false; }
    return true;
}

std::vector<std::vector<int>> palindromePairs(std::vector<std::string>& words) {
    std::unordered_map<std::string, int> wordMap;
    for (int i = 0; i < (int)words.size(); i++) wordMap[words[i]] = i;

    std::vector<std::vector<int>> result;

    for (int i = 0; i < (int)words.size(); i++) {
        const std::string& w = words[i];
        int n = w.size();

        for (int cut = 0; cut <= n; cut++) {
            // Case 1 & 2: prefix = w[0..cut-1], suffix = w[cut..n-1]
            // If suffix is palindrome, look for reverse of prefix
            if (isPalin(w, cut, n - 1)) {
                std::string revPrefix = std::string(w.begin(), w.begin() + cut);
                std::reverse(revPrefix.begin(), revPrefix.end());
                auto it = wordMap.find(revPrefix);
                if (it != wordMap.end() && it->second != i) {
                    result.push_back({i, it->second});  // words[i] + revPrefix
                }
            }
            // Case 3: prefix = w[0..cut-1], if prefix is palindrome,
            // look for reverse of suffix
            if (cut > 0 && isPalin(w, 0, cut - 1)) {
                std::string revSuffix = std::string(w.begin() + cut, w.end());
                std::reverse(revSuffix.begin(), revSuffix.end());
                auto it = wordMap.find(revSuffix);
                if (it != wordMap.end() && it->second != i) {
                    result.push_back({it->second, i});  // revSuffix + words[i]
                }
            }
        }
    }
    return result;
}
// Time: O(n * L²) -- n words, L cuts per word, L for isPalin check
// Better than O(n²L) naive when L << n
```

---

## 2. Map Sum Pairs (LC 677)

**Why tricky**: Insert `(key, val)`. Query sum of all values whose keys start with prefix. A trie with cumulative sum at each node requires careful update on re-insert (if key already exists, delta = new_val - old_val).

```cpp
class MapSum {
    struct MSNode {
        MSNode* children[26];
        int val;       // stored value at this word (0 if not end of word)
        int sum;       // sum of all values in subtree rooted here
        MSNode() : val(0), sum(0) {
            for (int i = 0; i < 26; i++) children[i] = nullptr;
        }
    };
    MSNode* root;
    std::unordered_map<std::string, int> keyMap;  // track existing key values

public:
    MapSum() { root = new MSNode(); }

    void insert(std::string key, int val) {
        int delta = val - keyMap[key];  // if key already exists, only add delta
        keyMap[key] = val;
        MSNode* cur = root;
        for (char c : key) {
            int i = c - 'a';
            if (!cur->children[i]) cur->children[i] = new MSNode();
            cur = cur->children[i];
            cur->sum += delta;  // propagate delta through path
        }
        cur->val = val;
    }

    int sum(std::string prefix) {
        MSNode* cur = root;
        for (char c : prefix) {
            int i = c - 'a';
            if (!cur->children[i]) return 0;
            cur = cur->children[i];
        }
        return cur->sum;  // sum of all words with this prefix
    }
};
// Insert: O(L), Sum: O(L) -- both optimal
```

---

## 3. Maximum XOR of Two Numbers with Queries (LC 1707)

**Why tricky**: Given nums[] and queries[xi, mi], for each query find max XOR of xi with any element in nums where element <= mi. Offline: sort queries and nums by mi, add elements to XOR trie incrementally.

```cpp
int maximumXorWithQueries(std::vector<int>& nums, std::vector<std::vector<int>>& queries) {
    // Sort queries offline by mi (threshold)
    int Q = queries.size();
    std::vector<int> qIdx(Q);
    std::iota(qIdx.begin(), qIdx.end(), 0);  // C++17: fill 0,1,2,...,Q-1
    std::sort(qIdx.begin(), qIdx.end(), [&](int a, int b) {
        return queries[a][1] < queries[b][1];
    });
    std::sort(nums.begin(), nums.end());

    XorTrie trie;  // from Pattern 5 in 01_Patterns.md
    std::vector<int> answers(Q, -1);
    int ni = 0;

    for (int qi : qIdx) {
        int x = queries[qi][0], m = queries[qi][1];
        // Add all nums <= m to trie
        while (ni < (int)nums.size() && nums[ni] <= m) {
            trie.insert(nums[ni++]);
        }
        if (ni > 0) answers[qi] = trie.maxXOR(x);
    }
    return *std::max_element(answers.begin(), answers.end());
}
// Time: O((n + Q) log(max_value)), Space: O(n * 32)
// std::iota: fills range with sequentially increasing values -- C++11
```

---

## 4. Count Distinct Substrings Using Suffix Trie

**Why tricky**: Every substring is a prefix of some suffix. Insert all suffixes into a trie. Count of distinct substrings = total number of nodes in the trie (each node = one unique character extending a unique prefix).

```cpp
int countDistinctSubstrings(const std::string& s) {
    // Insert all suffixes
    Trie trie;
    int distinctCount = 0;

    // Instead of counting nodes, count insertions of NEW nodes
    // Modified insert that returns new nodes added
    struct CountingTrie {
        TrieNode* root;
        CountingTrie() { root = new TrieNode(); }

        int insertAndCount(const std::string& s, int start) {
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

*<- [[08_Tries_Problems_and_Exercises|Problems]] · [[../09_Heap_Priority_Queue/00_Index|Heap ->]]*
