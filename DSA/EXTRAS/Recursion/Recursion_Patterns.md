---
tags: [dsa, recursion, patterns, tail-recursion, head-recursion]
links: ["[[00_Index]]", "[[Recursion_Problems]]"]
---

# Recursion -- Patterns

*<- [[00_Index|Index]] · [[Recursion_Problems|Problems ->]]*

---

## What is Recursion?

Recursion is a process in which a function calls itself directly or indirectly. 
- **Base Case**: The stopping condition that terminates recursion and prevents infinite loops (stack overflow).
- **Recurrence Relation / Step**: The logic that reduces the problem size and calls the function again.

---

## Four Main Types of Recursion

### 1. Tail Recursion
The recursive call is the **very last statement** executed in the function. There are no operations left to do after the call returns.
- **Optimization**: Modern compilers can optimize tail-recursive functions into simple iterative loops (Tail Call Optimization - TCO) to save stack space.

```cpp
// Prints from n down to 1
void tailPrint(int n) {
    if (n == 0) return; // Base case
    cout << n << " ";
    tailPrint(n - 1);   // Tail recursive call
}
```

```
Stack trace: tailPrint(3) -> prints 3 -> tailPrint(2) -> prints 2 -> tailPrint(1) -> prints 1 -> returns.
```

---

### 2. Head Recursion
The recursive call is made at the **beginning** of the function, before any other operations are performed. All operations happen on the **way back up** the call stack.

```cpp
// Prints from 1 up to n
void headPrint(int n) {
    if (n == 0) return;
    headPrint(n - 1);   // Head recursive call
    cout << n << " ";   // Operation happens after returning
}
```

```
Stack Frame Trace:
- headPrint(3) suspends -> calls headPrint(2)
  - headPrint(2) suspends -> calls headPrint(1)
    - headPrint(1) suspends -> calls headPrint(0) (returns)
  - headPrint(1) prints 1
- headPrint(2) prints 2
- headPrint(3) prints 3
```

---

### 3. Tree Recursion
The function makes **more than one self-call** in its body. This forms a tree-like branching execution trace (e.g. Fibonacci, merge sort).

```cpp
void treeRecursion(int n) {
    if (n <= 0) return;
    cout << n << " ";
    treeRecursion(n - 1); // Branch 1
    treeRecursion(n - 2); // Branch 2
}
```

```
Recursion Tree for n = 3:
                     tree(3)
                    /       \
               tree(2)      tree(1)
               /     \      /     \
           tree(1)  tree(0) tree(0) tree(-1)
```

---

### 4. Nested Recursion
A function passes a **recursive call of itself as a parameter** to another recursive call (e.g. Ackermann function, MacCarthy 91 function).

```cpp
int nestedRec(int n) {
    if (n > 100) return n - 10;
    return nestedRec(nestedRec(n + 11)); // nested parameter
}
```

---

*<- [[00_Index|Index]] · [[Recursion_Problems|Problems ->]]*
