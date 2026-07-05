---
tags: [cpp, core, value-categories, lvalue, rvalue, prvalue, xvalue]
links: ["[[02_Core_Index]]", "[[02_Core_Const_Constexpr_Consteval]]"]
---

# Core Language Mechanics -- Value Categories

*<- [[02_Core_Index|Index]] · [[02_Core_Const_Constexpr_Consteval|Const & Compile-Time ->]]*

---

## 1. Expression Value Taxonomy

In C++, **value categories** are properties of **expressions**, not of objects. An object is a region of storage; an expression is a syntactic construct that evaluates to some value/object and has a value category.

Since C++11, expressions are categorized based on two primary properties:
1. **Has Identity**: The expression refers to an object with a persistent memory address (so you can take its address with `&`).
2. **Movable (Can be resource-looted / expired)**: The expression represents an object that can be safely moved from because it is temporary.

```
                         Expression (glvalue)
                       /                      \
             lvalue (identity)                 rvalue
                                              /      \
                             xvalue (expiry)          prvalue (pure rvalue)
```

### The Five Categories:
- **`glvalue` (generalized lvalue)**: Has identity.
- **`lvalue`**: Has identity, but cannot be moved from (ordinary variables, named references).
- **`rvalue`**: Can be moved from (temporaries, cast-to-rvalue-reference results).
- **`prvalue` (pure rvalue)**: Has no identity, but can be moved from (literals, temporary return values of operations).
- **`xvalue` (eXpiring value)**: Has identity AND can be moved from (e.g., the result of `std::move(x)`).

---

## 2. Structural Property Matrix

| Category | Has Identity? | Movable? | Example Expressions |
|---|---|---|---|
| **`lvalue`** | **Yes** | No | `x` (named variable), `arr[i]`, `*ptr`, `obj.member` |
| **`xvalue`** | **Yes** | **Yes** | `std::move(x)`, `static_cast<T&&>(x)` |
| **`prvalue`**| No | **Yes** | `42` (literal), `a + b` (temp arithmetic result), `func()` returning value |

---

## 3. How Value Categories Underpin Move Semantics

The compiler decides whether to invoke a copy constructor `T(const T&)` or a move constructor `T(T&&)` by inspecting the value category of the argument expression:
- If the argument is an **`rvalue`** (either `prvalue` or `xvalue`), it matches the move constructor overload.
- If the argument is an **`lvalue`**, it matches the copy constructor.

### The Named Parameter Pitfall

A very common source of confusion is that **rvalue references themselves can be lvalues**. 
- If a variable has a **name**, the expression referring to it is an **lvalue**, regardless of its declared type!

```cpp
#include <utility>
#include <vector>

void process(std::vector<int>& lref) {
    // Called for lvalues
}

void process(std::vector<int>&& rref) {
    // Called for rvalues
}

void handle(std::vector<int>&& vec) {
    // 'vec' is an rvalue reference type.
    // However, because 'vec' has a name, the expression 'vec' is an LVALUE!
    
    process(vec); // Invokes process(std::vector<int>&) -> Copy semantics!
    
    // To cast it back to a movable xvalue, we must strip its name/identity using std::move
    process(std::move(vec)); // Invokes process(std::vector<int>&&) -> Move semantics!
}
```

---

*<- [[02_Core_Index|Index]] · [[02_Core_Const_Constexpr_Consteval|Const & Compile-Time ->]]*
