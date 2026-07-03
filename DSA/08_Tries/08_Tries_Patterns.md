---
tags: [dsa, tries, trie, patterns, xor-trie]
links: ["[[08_Tries_Index]]", "[[08_Tries_Problems_and_Exercises]]"]
---

# Tries -- Patterns

*<- [[08_Tries_Index\|Index]] · [[08_Tries_Problems_and_Exercises\|Problems ->]]*

---

## What is a Trie?

A Trie (prefix tree) stores strings character by character. Each node represents one character. All strings sharing a common prefix share the same path from root.

```
Insert: "cat", "car", "cart", "dog"

        root
       /    \
      c      d
      |      |
      a      o
     / \     |
    t   r    g(*)
   (*)  |
        t(*)
        |
        (*)

(*) = end of word marker

"ca" is a prefix of "cat" and "car"
"car" is a prefix of "cart"
```

**Why Trie over HashMap**: Trie enables prefix-based searches in O(L) time where L = word length. Hash map can only check exact strings in O(L), not all words with a given prefix.

---

## Pattern 1: Basic Trie -- Insert, Search, StartsWith

### Approach A: Fixed Array (26 children for lowercase letters)

```cpp
struct TrieNode {
    TrieNode* children[26];
    bool isEnd;

    TrieNode() : isEnd(false) {
        for (int i = 0; i < 26; i++) children[i] = nullptr;
    }
};

class Trie {
public:
    TrieNode* root;

    Trie() { root = new TrieNode(); }

    // Insertion Dry Run for "cat":
    // 1. Start at root. c - 'a' = 2. children[2] is null -> create new TrieNode. Move to it.
    // 2. c - 'a' = 0. children[0] is null -> create new TrieNode. Move to it.
    // 3. t - 'a' = 19. children[19] is null -> create new TrieNode. Move to it.
    // 4. Word ends: set cur->isEnd = true.
    void insert(const string& word) {
        TrieNode* cur = root;
        for (char c : word) {
            int idx = c - 'a';
            if (!cur->children[idx]) {
                cur->children[idx] = new TrieNode();
            }
            cur = cur->children[idx];
        }
        cur->isEnd = true;
    }

    bool search(const string& word) {
        TrieNode* cur = root;
        for (char c : word) {
            int idx = c - 'a';
            if (!cur->children[idx]) return false;
            cur = cur->children[idx];
        }
        return cur->isEnd;  // must be end of word, not just a prefix
    }

    bool startsWith(const string& prefix) {
        TrieNode* cur = root;
        for (char c : prefix) {
            int idx = c - 'a';
            if (!cur->children[idx]) return false;
            cur = cur->children[idx];
        }
        return true;  // just need to reach end of prefix -- don't check isEnd
    }

    // MEMORY DELETION / GARBAGE COLLECTION (Classic Interview Addition)
    // Why tricky: We must delete unused nodes bottom-up (post-order DFS).
    // A node can only be deleted if:
    // 1. It has no children (all children are nullptr).
    // 2. It is not marked as the end of another word (isEnd == false).
    bool deleteWord(TrieNode* curr, const string& word, int depth) {
        if (!curr) return false;

        // Base case: Reached the end of the word
        if (depth == (int)word.size()) {
            if (!curr->isEnd) return false; // word doesn't exist
            curr->isEnd = false;            // unmark end of word
            return isEmpty(curr);           // if node has no children, it can be deleted
        }

        int idx = word[depth] - 'a';
        bool shouldDeleteChild = deleteWord(curr->children[idx], word, depth + 1);

        if (shouldDeleteChild) {
            delete curr->children[idx];
            curr->children[idx] = nullptr;

            // Recurse upward: curr can be deleted if it has no other children
            // AND it is not the end of another word
            return !curr->isEnd && isEmpty(curr);
        }
        return false;
    }

private:
    bool isEmpty(TrieNode* node) {
        for (int i = 0; i < 26; i++) {
            if (node->children[i]) return false;
        }
        return true;
    }
};
// Insert/Search/StartsWith/Delete: O(L) time, O(L) space per insertion
// Total space: O(ALPHABET * total_characters) in worst case
```

### Pointer Dry Run Diagram: Inserting "car" after "cat"
```
               root
                |
              children[2] ('c')
                |
              children[0] ('a')
             /          \
  children[19] ('t')   children[17] ('r')  <-- newly created node
     [isEnd=true]         [isEnd=true]
```

### Approach B: HashMap for children (better when alphabet is large or sparse)

```cpp
struct TrieNodeMap {
    unordered_map<char, TrieNodeMap*> children;
    bool isEnd = false;
};
// Use when: Unicode characters, arbitrary symbols, or when words are sparse
// Tradeoff: more cache misses than array approach, but less wasted space
```

---

## Pattern 2: Count Prefixes and Words

```cpp
// Extended TrieNode tracking counts -- useful for autocomplete and deletion
struct TrieNodeCount {
    TrieNodeCount* children[26];
    int wordCount;    // how many complete words end here
    int prefixCount;  // how many words pass through here

    TrieNodeCount() : wordCount(0), prefixCount(0) {
        for (int i = 0; i < 26; i++) children[i] = nullptr;
    }
};

class TrieWithCount {
    TrieNodeCount* root;
public:
    TrieWithCount() { root = new TrieNodeCount(); }

    void insert(const string& word) {
        TrieNodeCount* cur = root;
        for (char c : word) {
            int idx = c - 'a';
            if (!cur->children[idx]) cur->children[idx] = new TrieNodeCount();
            cur = cur->children[idx];
            cur->prefixCount++;
        }
        cur->wordCount++;
    }

    int countWordsEqualTo(const string& word) {
        TrieNodeCount* cur = root;
        for (char c : word) {
            int idx = c - 'a';
            if (!cur->children[idx]) return 0;
            cur = cur->children[idx];
        }
        return cur->wordCount;
    }

    int countWordsStartingWith(const string& prefix) {
        TrieNodeCount* cur = root;
        for (char c : prefix) {
            int idx = c - 'a';
            if (!cur->children[idx]) return 0;
            cur = cur->children[idx];
        }
        return cur->prefixCount;
    }

    void erase(const string& word) {
        if (countWordsEqualTo(word) == 0) return;  // word doesn't exist
        TrieNodeCount* cur = root;
        for (char c : word) {
            int idx = c - 'a';
            cur->children[idx]->prefixCount--;
            cur = cur->children[idx];
        }
        cur->wordCount--;
    }
};
```

---

## Pattern 3: Longest Common Prefix (LC 14)

```cpp
// Insert all words into trie. Then walk from root as long as:
// 1. Current node has exactly one child, AND
// 2. Current node is not an end of word

string longestCommonPrefix(vector<string>& strs) {
    if (strs.empty()) return "";
    Trie trie;
    for (const string& s : strs) trie.insert(s);

    string prefix = "";
    TrieNode* cur = trie.root;  // accessing root directly (make it public or add method)

    for (char c : strs[0]) {
        int idx = c - 'a';
        // Count how many children the current node has
        int childCount = 0;
        for (int i = 0; i < 26; i++) if (cur->children[i]) childCount++;

        if (childCount == 1 && !cur->isEnd) {
            prefix += c;
            cur = cur->children[idx];
        } else {
            break;
        }
    }
    return prefix;
}
// Simpler approach: sort strings, compare first and last -- O(n log n + L)
// Trie approach: O(n*L) build + O(L) query
```

---

## Pattern 4: Word Search II -- Trie + DFS on Grid (LC 212)

**Why Trie is needed**: Find all words from a dictionary that appear in a 2D grid. Naive: run word search for each word separately -- O(words * grid * 4^word_length). With Trie: build dictionary into trie, run one DFS, explore all words simultaneously.

```cpp
struct WordTrieNode {
    WordTrieNode* children[26];
    string word;  // store full word at terminal node instead of bool
    WordTrieNode() : word("") {
        for (int i = 0; i < 26; i++) children[i] = nullptr;
    }
};

class Solution {
    void dfs(vector<vector<char>>& board, int r, int c,
             WordTrieNode* node, vector<string>& result) {
        char ch = board[r][c];
        if (ch == '#') return;  // already visited
        int idx = ch - 'a';
        if (!node->children[idx]) return;  // no word in trie with this prefix

        node = node->children[idx];
        if (!node->word.empty()) {
            result.push_back(node->word);
            node->word = "";  // avoid duplicates -- mark as found
        }

        board[r][c] = '#';  // mark visited
        int dirs[4][2] = {{0,1},{0,-1},{1,0},{-1,0}};
        for (auto& d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr >= 0 && nr < (int)board.size() &&
                nc >= 0 && nc < (int)board[0].size()) {
                dfs(board, nr, nc, node, result);
            }
        }
        board[r][c] = ch;  // restore
    }

public:
    vector<string> findWords(
        vector<vector<char>>& board,
        vector<string>& words
    ) {
        WordTrieNode* root = new WordTrieNode();
        // Build trie
        for (const string& w : words) {
            WordTrieNode* cur = root;
            for (char c : w) {
                int idx = c - 'a';
                if (!cur->children[idx]) cur->children[idx] = new WordTrieNode();
                cur = cur->children[idx];
            }
            cur->word = w;
        }

        vector<string> result;
        int rows = board.size(), cols = board[0].size();
        for (int r = 0; r < rows; r++)
            for (int c = 0; c < cols; c++)
                dfs(board, r, c, root, result);

        return result;
    }
};
// Time: O(M * 4 * 3^(L-1)) where M = grid cells, L = max word length
// The 3 (not 4) is because we never go back to where we came from
```

---

## Pattern 5: XOR Trie (Maximum XOR of Two Numbers -- LC 421)

**Why a bit trie**: To maximize XOR, for each bit of a number we want the opposite bit in the other number. A binary trie (bits from MSB to LSB) lets us greedily pick the opposite bit at each level.

```
XOR is maximized when bits differ.
For each number, traverse bit trie from MSB (bit 31) to LSB (bit 0).
At each level, try to go to the opposite bit. If not possible, go same.

Example: nums = [3, 10, 5, 25, 2, 8]
Binary: 3=00011, 10=01010, 5=00101, 25=11001

For 25 (11001), find number giving max XOR:
bit 4 of 25 = 1, want 0: go left. 5 (00101) starts with 0 -> possible.
bit 3 of 25 = 1, want 0: 5's bit 3 = 0. Go left.
...
25 XOR 5 = 28 = 11100 in 5 bits
```

```cpp
struct XorTrieNode {
    XorTrieNode* children[2];  // 0 and 1 only
    XorTrieNode() : children{nullptr, nullptr} {}
};

class XorTrie {
    XorTrieNode* root;

public:
    XorTrie() { root = new XorTrieNode(); }

    void insert(int num) {
        XorTrieNode* cur = root;
        for (int bit = 31; bit >= 0; bit--) {
            int b = (num >> bit) & 1;
            if (!cur->children[b]) cur->children[b] = new XorTrieNode();
            cur = cur->children[b];
        }
    }

    int maxXOR(int num) {
        XorTrieNode* cur = root;
        int result = 0;
        for (int bit = 31; bit >= 0; bit--) {
            int b = (num >> bit) & 1;
            int want = 1 - b;  // we want the opposite bit for max XOR
            if (cur->children[want]) {
                result |= (1 << bit);   // this bit contributes to XOR
                cur = cur->children[want];
            } else {
                cur = cur->children[b];  // forced to take same bit
            }
        }
        return result;
    }
};

int findMaximumXOR(vector<int>& nums) {
    XorTrie trie;
    for (int x : nums) trie.insert(x);
    int maxVal = 0;
    for (int x : nums) maxVal = max(maxVal, trie.maxXOR(x));
    return maxVal;
}
// Time: O(n * 32) = O(n), Space: O(n * 32) = O(n)
```

---

## Pattern 6: Replace Words (LC 648)

```cpp
// Given dictionary of roots, replace each word in sentence with its shortest root
string replaceWords(vector<string>& dictionary, string sentence) {
    Trie trie;
    for (const string& root : dictionary) trie.insert(root);

    istringstream iss(sentence);
    string word, result = "";
    while (iss >> word) {
        if (!result.empty()) result += " ";
        // Find shortest prefix in trie
        TrieNode* cur = trie.root;
        string replacement = "";
        bool found = false;
        for (char c : word) {
            int idx = c - 'a';
            if (!cur->children[idx]) break;
            cur = cur->children[idx];
            replacement += c;
            if (cur->isEnd) { found = true; break; }
        }
        result += found ? replacement : word;
    }
    return result;
}
// Time: O(sum of lengths in dictionary + sentence length)
```

---

*<- [[08_Tries_Index\|Index]] · [[08_Tries_Problems_and_Exercises\|Problems ->]]*
