---
tags: [cpp, performance, move-semantics, emplace-back, push-back, pass-by-value]
links: ["[[09_Perf_Index]]", "[[09_Perf_Memory_Alignment_and_Padding]]", "[[09_Perf_Small_Object_Optimizations]]"]
---

# Performance & Optimization -- Move Semantics for Performance

*<- [[09_Perf_Memory_Alignment_and_Padding|Memory Alignment & Padding]] · [[09_Perf_Small_Object_Optimizations|Small Object Optimizations ->]]*

---

Move semantics optimize performance by substituting deep-copy allocations with cheap pointer swaps.

---

## 1. Pass-by-Value + `std::move` (The Constructor Pattern)

When writing functions or constructors that sink (store) their parameters, you can use a single overload that accepts parameters **by value** and then moves them.

- If the caller passes an **lvalue**, it is copied into the parameter, then moved into the destination (1 copy, 1 move).
- If the caller passes an **rvalue**, it is moved into the parameter, then moved into the destination (0 copies, 2 moves).
- This is cleaner and more optimal than maintaining multiple lvalue-reference and rvalue-reference overloads.

```cpp
#include <string>
#include <utility>

class User {
    std::string name;
public:
    // Single constructor accepting std::string by value
    explicit User(std::string n) : name(std::move(n)) {}
};

void demoUser() {
    std::string name = "Alice";
    User u1(name);            // Lvalue passed: 1 copy, 1 move
    User u2(std::move(name)); // Rvalue passed: 0 copies, 2 moves
}
```

---

## 2. `push_back` vs `emplace_back` (In-Place Construction)

When appending elements to a `std::vector`, choosing the right insertion method alters allocation counts.

- **`push_back(const T&)` / `push_back(T&&)`**:
  - Requires a constructed object.
  - The object is copied or moved into the vector's internal array, and the temporary object is destroyed.
- **`emplace_back(Args&&...)`**:
  - Accepts the **arguments** for the object's constructor.
  - The vector constructs the object **directly in-place** inside its allocated memory using placement new.
  - This completely eliminates temporary objects and copy/move constructor calls!

```cpp
#include <vector>
#include <string>

struct Item {
    std::string title;
    int price;
    Item(std::string t, int p) : title(std::move(t)), price(p) {}
};

void demoInsertion() {
    std::vector<Item> items;
    
    // 1. push_back: creates a temporary Item, moves it, then destroys the temporary
    items.push_back(Item("Book", 20)); 
    
    // 2. emplace_back: constructs the Item directly in the vector's memory block!
    // No temporary Item is ever constructed or moved.
    items.emplace_back("Book", 20); 
}
```

---

*<- [[09_Perf_Memory_Alignment_and_Padding|Memory Alignment & Padding]] · [[09_Perf_Small_Object_Optimizations|Small Object Optimizations ->]]*
