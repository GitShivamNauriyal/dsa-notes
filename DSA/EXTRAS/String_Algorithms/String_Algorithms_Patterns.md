---
tags: [dsa, strings, kmp, rabin-karp, rolling-hash, lps]
links: ["[[00_Index]]", "[[String_Algorithms_Problems]]"]
---

# String Algorithms -- Substring Matching

*<- [[00_Index|Index]] · [[String_Algorithms_Problems|Problems ->]]*

---

## 1. Knuth-Morris-Pratt (KMP) Algorithm

**The Problem**: Find all occurrences of pattern `pat` in text `txt` in $O(N + M)$ time.
- **Why Naive fails**: If `txt = "aaaaaab"`, `pat = "aab"`, matching naively resets the pointer back to start, taking $O(N \times M)$ time.
- **KMP Key Idea**: Never backtrack the text pointer. When a mismatch occurs, use the **LPS array** to skip redundant comparisons.

### The LPS Array (Longest Proper Prefix which is also Suffix)
`lps[i]` stores the length of the longest proper prefix of `pat[0..i]` that is also a suffix of `pat[0..i]`.
- For `pat = "aabaaba"`:
  - `i = 0`: "a" -> 0
  - `i = 1`: "aa" -> prefix "a", suffix "a" -> 1
  - `i = 2`: "aab" -> 0
  - `i = 3`: "aaba" -> prefix "a", suffix "a" -> 1
  - `i = 4`: "aabaa" -> prefix "aa", suffix "aa" -> 2
  - `i = 5`: "aabaab" -> prefix "aab", suffix "aab" -> 3
  - `i = 6`: "aabaaba" -> prefix "aaba", suffix "aaba" -> 4
  - `lps` array = `[0, 1, 0, 1, 2, 3, 4]`

```cpp
#include <string>
#include <vector>

using namespace std;

// Generate the LPS table
vector<int> computeLPSArray(const string& pat) {
    int m = pat.size();
    vector<int> lps(m, 0);
    int len = 0; // length of the previous longest prefix suffix
    int i = 1;

    while (i < m) {
        if (pat[i] == pat[len]) {
            len++;
            lps[i] = len;
            i++;
        } else {
            if (len != 0) {
                len = lps[len - 1]; // fallback to previous match
            } else {
                lps[i] = 0;
                i++;
            }
        }
    }
    return lps;
}

// KMP Search
vector<int> KMPSearch(const string& pat, const string& txt) {
    int m = pat.size();
    int n = txt.size();
    vector<int> lps = computeLPSArray(pat);
    vector<int> result;

    int i = 0; // index for txt
    int j = 0; // index for pat

    while (i < n) {
        if (pat[j] == txt[i]) {
            i++; j++;
        }

        if (j == m) {
            result.push_back(i - j); // found match at index i-j
            j = lps[j - 1];
        } else if (i < n && pat[j] != txt[i]) {
            // Mismatch after j matches
            if (j != 0) {
                j = lps[j - 1]; // skip comparisons using LPS
            } else {
                i++;
            }
        }
    }
    return result;
}
// Time Complexity: O(N + M)
// Space Complexity: O(M) for LPS array
```

---

## 2. Rabin-Karp Algorithm (Rolling Hash)

**Concept**: Compute a hash value of the pattern and each substring window of the text of length $M$. If the hashes match, perform a character-by-character comparison to confirm the match.

### Rolling Hash formula
We can slide the window and compute the next hash value in $O(1)$ time by deducting the leading character hash and adding the trailing character:
$$H_{i+1} = \left( d \cdot (H_i - \text{txt}[i] \cdot h) + \text{txt}[i+M] \right) \bmod q$$
- Where $d$ is the size of the alphabet (e.g. 256), $q$ is a large prime number to prevent overflows (e.g. 101), and $h = d^{M-1} \bmod q$.

```cpp
#include <string>
#include <vector>

using namespace std;

vector<int> RabinKarpSearch(const string& pat, const string& txt, int q = 101) {
    int m = pat.size();
    int n = txt.size();
    int d = 256; // alphabet base size
    int h = 1;   // stores d^(m-1) % q
    vector<int> result;

    for (int i = 0; i < m - 1; i++) {
        h = (h * d) % q;
    }

    int p = 0; // hash value for pattern
    int t = 0; // hash value for txt window

    // Calculate initial hashes
    for (int i = 0; i < m; i++) {
        p = (d * p + pat[i]) % q;
        t = (d * t + txt[i]) % q;
    }

    for (int i = 0; i <= n - m; i++) {
        // If hashes match, verify characters
        if (p == t) {
            bool match = true;
            for (int j = 0; j < m; j++) {
                if (txt[i + j] != pat[j]) {
                    match = false;
                    break;
                }
            }
            if (match) result.push_back(i);
        }

        // Slide window: compute next hash
        if (i < n - m) {
            t = (d * (t - txt[i] * h) + txt[i + m]) % q;
            if (t < 0) t += q; // normalize negative mod results
        }
    }
    return result;
}
// Time Complexity: O(N + M) average case, O(N * M) worst case (extreme hash collisions)
// Space Complexity: O(1)
```

---

*<- [[00_Index]] · [[String_Algorithms_Problems|Problems ->]]*
