# DSA Vault -- Implementation Guide for Continuation

> This document is for any AI models continuing this vault build.
> Read this ENTIRELY before writing a single file. These are hard rules.

---

## Context

This is an Obsidian vault at `c:\USER_DATA\SHIVAM\dat\vaults\vaults\SDE`.
The user is a student targeting SDE and Quant placements at top-tier companies (FAANG, quant firms, top Indian product companies).
The DSA notes go inside `DSA/` folder.
Git is initialized. Commit after each chapter completes.

---

## Behavior Rules (CRITICAL)

- **ZERO verbosity in chat.** Do not explain what you just did. Do not say "here is what I created". Do not use emojis. Do not use em-dashes. No sycophantic openers. No closing fluff.
- After every chapter commit, **stop and ask only "Continue?"** -- nothing else.
- **Token efficiency**: write content, not commentary about the content.
- User's exact words: *"dont waste tokens telling me what you are doing"*

---

## Content Rules

### Language
- **C++20 only.** No Python, no Java.
- For non-trivial C++20 features (`std::span`, `std::ranges`, structured bindings, `std::erase_if`, etc.), add a short comment explaining what it is and why it is used.
- Write the **simple/normal way first**, then show the modern C++ shorthand below it.
- Avoid overly complex syntax that would confuse a junior (no `int i(0uz)` style C++23 things).

### Structure Per Chapter
Every chapter folder must contain these files in order:

```
00_Index.md                 <- table of files, nav links
01_Foundations.md           <- (for complex topics like Trees, LL) basics first
01_Patterns.md OR 02_Patterns.md  <- all patterns with full C++ code
02/03_Problems_and_Exercises.md   <- 4-tier problem table + 2 worked solutions
LAST_Tricky.md              <- hard/higher-order interview questions (REQUIRED for every chapter)
```

### Note Format Per File
1. YAML frontmatter with `tags` and `links`
2. Navigation links at top and bottom: `*<- [[prev]] · [[next ->]]*`
3. Trivial explanation FIRST, complex logic AFTER with "why" reasoning
4. Every pattern: explain visually with ASCII walkthrough, then full C++ code
5. Every code block: labeled with the problem source if applicable

### Tricky File (MANDATORY for every chapter)
This is the most important file per chapter. Contains:
- Problems that are NOT obvious applications of the chapter's patterns
- Higher-order thinking: e.g., Binary Search on an infinite array, Graphs with multiple sources BFS, DP on a tree, etc.
- Interview-level insights that distinguish good from great candidates
- Each problem has a "Why tricky" section BEFORE the solution
- Problem table at the end linking to LC/GFG/Codeforces

### Exercises Format
4 tiers, always:
- **Tier 1**: Foundations (all easy, solve all)
- **Tier 2**: Core patterns (medium, solve all)
- **Tier 3**: Interview level (medium-hard, solve at least 5)
- **Tier 4**: Placement-hard / OA-level (hard, do what you can)

Difficulty tags: `Easy`, `Medium`, `Hard` (LeetCode style).
Every problem must have: problem number + platform link.
Preferred platforms: LeetCode > GFG > Codeforces.

### Obsidian Graph View
- Every file has YAML `tags` and `links` arrays
- Navigation links `[[prev]]` and `[[next]]` at top and bottom of every file
- Index files link to all sub-files in the chapter
- `00_Home.md` links to all chapter indexes

---

## Completed Chapters (Do NOT redo these)

| # | Chapter | Files | Status |
|---|---------|-------|--------|
| 0 | `00_Complexity_Cheatsheet.md` | Big-O, Master Theorem (6 examples), STL complexities, amortized | DONE |
| 1 | `01_Arrays_and_Hashing/` | Basics, Prefix Sums, Hashing+Rolling Hash, Key Algorithms (Kadane/DNF/Boyer-Moore), Problems, Tricky | DONE |
| 2 | `02_Two_Pointers/` | Patterns (5 patterns), Problems, Tricky | DONE |
| 3 | `03_Sliding_Window/` | Patterns (fixed/variable/char/deque), Problems, Tricky | DONE |
| 4 | `04_Stack/` | Patterns (bracket/monotonic/expression/min-stack), Problems, Tricky | DONE |
| 5 | `05_Binary_Search/` | Patterns (7 patterns), Problems, Tricky (infinite array, fractional, 2D matrix, parallel BS) | DONE |
| 6 | `06_Linked_List/` | Foundations, Patterns (reversal/fast-slow/merge/nth/palindrome/intersection/sort), Problems (LRU), Tricky | DONE |
| 7 | `07_Trees/` | Foundations (all traversals recursive+iterative+BFS), BST, Patterns (LCA/path sum/construct/views/serialize), Problems, Tricky (Morris/segment tree/binary lifting) | DONE |

---

## Pending Chapters (Build in this order)

### Chapter 8: Tries (`08_Tries/`)
Files: `08_Tries_Index.md`, `08_Tries_Patterns.md`, `08_Tries_Problems_and_Exercises.md`, `08_Tries_Tricky.md`

Key patterns to cover:
- TrieNode structure (array[26] vs unordered_map for children)
- Insert, Search, StartsWith
- Word Search II (Trie + DFS on grid)
- Longest Common Prefix
- XOR Trie (maximum XOR of two numbers -- bit trie)
- Count distinct substrings using Trie
- Tricky: Palindrome pairs (LC 336), Replace words (LC 648), Map Sum pairs (LC 677)

---

### Chapter 9: Heap / Priority Queue (`09_Heap_Priority_Queue/`)
Files: `09_Heap_Priority_Queue_Index.md`, `09_Heap_Priority_Queue_Patterns.md`, `09_Heap_Priority_Queue_Problems_and_Exercises.md`, `09_Heap_Priority_Queue_Tricky.md`

Key patterns:
- `std::priority_queue` (max-heap default, min-heap with `greater<>`)
- Top K elements pattern
- K-way merge (generalization of merge k sorted lists)
- Median of a stream (two heaps: max-heap for lower half, min-heap for upper half)
- Task scheduler
- Tricky: K closest points, reorganize string, find median from data stream (LC 295), IPO (LC 502)

---

### Chapter 10: Backtracking (`10_Backtracking/`)
Files: `10_Backtracking_Index.md`, `10_Backtracking_Patterns.md`, `10_Backtracking_Problems_and_Exercises.md`, `10_Backtracking_Tricky.md`

Key patterns:
- Subsets (power set)
- Permutations (with and without duplicates)
- Combinations
- N-Queens
- Sudoku Solver
- Word Search (DFS on grid)
- Palindrome Partitioning
- Tricky: Expression Add Operators (LC 282), Remove Invalid Parentheses (LC 301), Zuma Game (LC 488)

---

### Chapter 11: Graphs (`11_Graphs/`)
Files: `11_Graphs_Index.md`, `11_Graphs_Foundations.md`, `11_Graphs_BFS_DFS.md`, `11_Graphs_Shortest_Path.md`, `11_Graphs_Union_Find.md`, `11_Graphs_Topological_Sort.md`, `11_Graphs_Problems_and_Exercises.md`, `11_Graphs_Tricky.md`

Key patterns:
- Graph representation: adjacency list, adjacency matrix, edge list
- BFS (shortest path unweighted), DFS (connected components, cycle detection)
- Dijkstra (weighted shortest path, priority queue based)
- Bellman-Ford (negative weights)
- Floyd-Warshall (all-pairs shortest path)
- Union-Find (DSU) with path compression + union by rank
- Topological Sort (Kahn's BFS, DFS-based)
- Tricky: Multi-source BFS (01 matrix LC 542), Bipartite check, Strongly Connected Components (Kosaraju), Bridges and Articulation Points

---

### Chapter 12: Advanced Graphs (`12_Advanced_Graphs/`)
Files: `12_Advanced_Graphs_Index.md`, `12_Advanced_Graphs_MST.md`, `12_Advanced_Graphs_Network_Flow.md`, `12_Advanced_Graphs_SCC.md`, `12_Advanced_Graphs_Problems_and_Exercises.md`, `12_Advanced_Graphs_Tricky.md`

Key patterns:
- Minimum Spanning Tree: Kruskal (Union-Find) + Prim (Priority Queue)
- Network Flow: Ford-Fulkerson concept, max-flow min-cut theorem
- Strongly Connected Components: Kosaraju's algorithm
- Bridges and Articulation Points: Tarjan's algorithm
- Euler Path/Circuit
- Tricky: Swim in rising water (LC 778), Reconstruct itinerary (LC 332), Critical connections (LC 1192)

---

### Chapter 13: Dynamic Programming (`13_DP/`) -- EXPANDED STRUCTURE

This is the most important chapter. Use this exact subfolder structure:

```
13_DP/
├── 00_Index.md               <- links to all subfolders
├── 01_1D_DP/
│   ├── 00_Index.md
│   ├── 01_Fibonacci_Pattern.md     (Fibonacci, Climbing Stairs, House Robber)
│   ├── 02_Coin_Change.md           (Coin Change, Minimum Jumps)
│   └── 03_Problems.md
├── 02_2D_Grid_DP/
│   ├── 00_Index.md
│   ├── 01_Grid_Paths.md            (Unique Paths, Min Path Sum)
│   ├── 02_Grid_Obstacles.md
│   └── 03_Problems.md
├── 03_Knapsack/
│   ├── 00_Index.md
│   ├── 01_01_Knapsack.md           (0/1 Knapsack -- base of all DP)
│   ├── 02_Unbounded_Knapsack.md    (Coin Change as unbounded knapsack)
│   ├── 03_Subset_Sum.md            (Partition Equal Subset Sum LC 416)
│   ├── 04_Target_Sum.md            (LC 494)
│   └── 05_Problems.md
├── 04_DP_on_Strings/
│   ├── 00_Index.md
│   ├── 01_LCS.md                   (Longest Common Subsequence)
│   ├── 02_LIS.md                   (Longest Increasing Subsequence -- also O(n log n))
│   ├── 03_Edit_Distance.md         (LC 72)
│   ├── 04_Palindrome_DP.md         (Palindrome Partitioning, Longest Palindromic Subsequence)
│   └── 05_Problems.md
├── 05_DP_on_Stocks/
│   ├── 00_Index.md
│   ├── 01_Single_Transaction.md    (LC 121)
│   ├── 02_Infinite_Transactions.md (LC 122)
│   ├── 03_With_Cooldown.md         (LC 309)
│   ├── 04_With_Fee.md              (LC 714)
│   ├── 05_Two_Transactions.md      (LC 123)
│   ├── 06_K_Transactions.md        (LC 188)
│   └── 07_Problems.md
├── 06_DP_on_Trees/
│   ├── 00_Index.md
│   ├── 01_Tree_DP.md               (House Robber III, Diameter, Max Path Sum)
│   └── 02_Problems.md
├── 07_Partition_DP/
│   ├── 00_Index.md
│   ├── 01_MCM.md                   (Matrix Chain Multiplication -- archetype)
│   ├── 02_Burst_Balloons.md        (LC 312)
│   ├── 03_Strange_Printer.md       (LC 664)
│   └── 04_Problems.md
├── 08_Bitmask_DP/
│   ├── 00_Index.md
│   ├── 01_Bitmask_DP.md            (TSP, Assign Tasks)
│   └── 02_Problems.md
└── 09_Interval_DP/
    ├── 00_Index.md
    ├── 01_Interval_DP.md           (Stone Merge, Palindrome cost)
    └── 02_Problems.md
```

Each DP sub-file must:
1. Start with the TRIVIAL recursive solution first
2. Add memoization (top-down)
3. Convert to tabulation (bottom-up)
4. Optimize space if possible
5. Explain WHY the state is defined the way it is

---

### Chapter 14: Greedy (`14_Greedy/`)
Files: `14_Greedy_Index.md`, `14_Greedy_Patterns.md`, `14_Greedy_Problems_and_Exercises.md`, `14_Greedy_Tricky.md`

Key patterns: Activity Selection, Huffman Coding concept, Jump Game, Candy, Gas Station

---

### Chapter 15: Intervals (`15_Intervals/`)
Files: `15_Intervals_Index.md`, `15_Intervals_Patterns.md`, `15_Intervals_Problems_and_Exercises.md`, `15_Intervals_Tricky.md`

Key patterns: Merge Intervals, Insert Interval, Meeting Rooms, Employee Free Time

---

### Chapter 16: Bit Manipulation (`16_Bit_Manipulation/`)
Files: `16_Bit_Manipulation_Index.md`, `16_Bit_Manipulation_Patterns.md`, `16_Bit_Manipulation_Problems_and_Exercises.md`, `16_Bit_Manipulation_Tricky.md`

Key patterns: Basic ops, XOR tricks, bit counting, power of 2, subset enumeration via bits

---

### Chapter 17: Math & Geometry (`17_Math_and_Geometry/`)
Files: `17_Math_and_Geometry_Index.md`, `17_Math_and_Geometry_Patterns.md`, `17_Math_and_Geometry_Problems_and_Exercises.md`, `17_Math_and_Geometry_Tricky.md`

Key patterns: Sieve of Eratosthenes, GCD/LCM, modular arithmetic, fast power, nCr mod p, geometry basics

---

### Extras (`EXTRAS/`)

```
EXTRAS/
├── Recursion/           <- Aditya Verma style: recursion tree, all patterns
├── Sorting_Algorithms/  <- Bubble, Selection, Insertion, Merge, Quick, Heap, Counting, Radix
├── Matrix/              <- Spiral, Rotate 90, Set Zeroes, Search
├── String_Algorithms/   <- KMP, Z-algorithm, Manacher's, Suffix Array concept
└── Number_Theory/       <- Primes, Euler's totient, Chinese Remainder Theorem
```

---

## Git Convention

```
Commit message format: feat(dsa): complete [Topic Name] chapter
```

After each commit, stop and type only: **"Continue?"**

---

## Curriculum Sources

- **NeetCode.io roadmap** (the image attached by user): Arrays->Two Pointers->Stack->Binary Search->Sliding Window->Linked List->Trees->Tries->Heap->Backtracking->Graphs->Advanced Graphs->1D DP->2D DP->Greedy->Intervals->Bit Manipulation->Math
- **TakeUForward (Striver's SDE Sheet)**: extends NeetCode with harder variants, especially in DP and Graphs
- **Aditya Verma (YouTube)**: known for DP series (Knapsack variants as foundation for all DP, and all other patterns and variants) and Backtracking/Recursion style

The notes go BEYOND NeetCode into TakeUForward territory specifically for:
- DP (full 9-subtype structure above)
- Graph algorithms (SCC, bridges, articulation points)
- String algorithms (KMP, Z, Manacher)
- Number theory

---

## File Naming Convention

```
XX_Chapter_Name_Index.md
XX_Chapter_Name_Foundations.md  (if needed)
XX_Chapter_Name_BST.md          (for BST if needed)
XX_Chapter_Name_Patterns.md
XX_Chapter_Name_Problems_and_Exercises.md
XX_Chapter_Name_Tricky.md       (always the last file, e.g., XX_Chapter_Name_Tricky.md)
```

## YAML Frontmatter Template

```yaml
---
tags: [dsa, topic-name, sub-topic]
links: ["[[../00_Home]]", "[[prev_file]]", "[[next_file]]"]
---
```
