---
tags: [dsa, tries, problems, exercises]
links: ["[[08_Tries_Index]]", "[[08_Tries_Patterns]]", "[[08_Tries_Tricky]]"]
---

# Tries -- Problems & Exercises

*<- [[08_Tries_Patterns|Patterns]] · [[08_Tries_Tricky|Tricky ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Implement Trie | Medium | Basic insert/search/prefix | [LC 208](https://leetcode.com/problems/implement-trie-prefix-tree/) |
| 2 | Longest Common Prefix | Easy | Trie or sort+compare | [LC 14](https://leetcode.com/problems/longest-common-prefix/) |
| 3 | Search Suggestions System | Medium | Trie + DFS for suggestions | [LC 1268](https://leetcode.com/problems/search-suggestions-system/) |

## Tier 2 -- Core Patterns

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 4 | Maximum XOR of Two Numbers | Medium | XOR bit trie | [LC 421](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/) |
| 5 | Replace Words | Medium | Trie prefix matching | [LC 648](https://leetcode.com/problems/replace-words/) |
| 6 | Map Sum Pairs | Medium | Trie with sum values | [LC 677](https://leetcode.com/problems/map-sum-pairs/) |
| 7 | Implement Magic Dictionary | Medium | Trie + one char diff | [LC 676](https://leetcode.com/problems/implement-magic-dictionary/) |

## Tier 3 -- Interview Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 8 | Word Search II | Hard | Trie + DFS on grid | [LC 212](https://leetcode.com/problems/word-search-ii/) |
| 9 | Design Add and Search Words | Medium | Trie + wildcard DFS | [LC 211](https://leetcode.com/problems/design-add-and-search-words-data-structure/) |
| 10 | Count of Distinct Substrings | Medium | Trie all suffixes | [GFG](https://www.geeksforgeeks.org/count-distinct-substrings-string-using-suffix-trie/) |
| 11 | Maximum XOR with an Element from Array | Hard | Offline queries + XOR trie | [LC 1707](https://leetcode.com/problems/maximum-xor-with-an-element-from-array/) |

## Tier 4 -- Placement-Hard

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 12 | Palindrome Pairs | Hard | Trie + palindrome check | [LC 336](https://leetcode.com/problems/palindrome-pairs/) |
| 13 | Word Squares | Hard | Trie + backtracking | [LC 425](https://leetcode.com/problems/word-squares/) |
| 14 | Maximum Sum BST in Binary Tree | Hard | Trie concept extension | [LC 1373](https://leetcode.com/problems/maximum-sum-bst-in-binary-tree/) |

---

## Worked Solution: LC 211 -- Design Add and Search Words

**Key insight**: Normal search is O(L). Wildcard `.` requires trying all 26 children -- use DFS.

```cpp
class WordDictionary {
    struct WTrieNode {
        WTrieNode* children[26];
        bool isEnd;
        WTrieNode() : isEnd(false) {
            for (int i = 0; i < 26; i++) children[i] = nullptr;
        }
    };
    WTrieNode* root;

    bool dfs(WTrieNode* node, const string& word, int idx) {
        if (idx == (int)word.size()) return node->isEnd;
        char c = word[idx];
        if (c == '.') {
            for (int i = 0; i < 26; i++) {
                if (node->children[i] && dfs(node->children[i], word, idx + 1))
                    return true;
            }
            return false;
        }
        int i = c - 'a';
        return node->children[i] && dfs(node->children[i], word, idx + 1);
    }

public:
    WordDictionary() { root = new WTrieNode(); }

    void addWord(string word) {
        WTrieNode* cur = root;
        for (char c : word) {
            int i = c - 'a';
            if (!cur->children[i]) cur->children[i] = new WTrieNode();
            cur = cur->children[i];
        }
        cur->isEnd = true;
    }

    bool search(string word) { return dfs(root, word, 0); }
};
```

---

## Worked Solution: LC 1268 -- Search Suggestions System

```cpp
// Insert all products, then for each prefix of searchWord,
// collect up to 3 lexicographically smallest matches via DFS
vector<vector<string>> suggestedProducts(
    vector<string>& products, string searchWord
) {
    Trie trie;  // extended with DFS collect method
    for (const string& p : products) trie.insert(p);

    // Simpler approach: sort + binary search (O(n log n + L * log n))
    sort(products.begin(), products.end());
    vector<vector<string>> result;
    string prefix = "";

    for (char c : searchWord) {
        prefix += c;
        auto it = lower_bound(products.begin(), products.end(), prefix);
        vector<string> suggestions;
        for (int i = 0; i < 3 && it != products.end(); i++, ++it) {
            // Check if this product still starts with prefix
            if (it->substr(0, prefix.size()) == prefix)
                suggestions.push_back(*it);
            else break;
        }
        result.push_back(suggestions);
    }
    return result;
}
// Time: O(n log n + L * log n), Space: O(1) extra (or O(n*L) for trie approach)
```

---

*<- [[08_Tries_Patterns|Patterns]] · [[08_Tries_Tricky|Tricky ->]]*
