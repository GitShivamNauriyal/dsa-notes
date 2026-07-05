---
tags: [cpp, templates, tricky, dependent-names, expression-templates]
links: ["[[04_Tmpl_Index]]", "[[04_Tmpl_Solutions]]", "[[../05_STL_and_Modern_Features/05_STL_Index]]"]
---

# Templates & Generics -- Tricky & Higher-Order

*<- [[04_Tmpl_Solutions|Solutions]] · [[../05_STL_and_Modern_Features/05_STL_Index|STL & Modern Features ->]]*

---

## 1. Dependent Names & `typename`

**The Scenario**: Accessing a nested type (like an iterator) inside a template class.

```cpp
#include <vector>

template <typename T>
void iterateContainer(const T& container) {
    // Intention: Declare an iterator of container type T.
    // Reality: COMPILER ERROR: dependent name 'T::const_iterator' is not parsed as a type!
    // T::const_iterator it = container.begin();
}
```

### Why it occurs (Two-Phase Lookup):
Under C++ two-phase lookup:
1. **Phase 1 (Parsing)**: The compiler checks syntax without knowing the template argument `T`. At this stage, it doesn't know if `T::const_iterator` is a nested type or a static variable. By default, it assumes it is a static variable (meaning `T::const_iterator * x` would be parsed as a multiplication expression, not a pointer declaration).
2. **Phase 2 (Instantiation)**: Compiler generates code for concrete `T`.

### The Fix:
You must explicitly prepend **`typename`** to tell the compiler that the nested dependent name is a type.

```cpp
template <typename T>
void iterateContainer(const T& container) {
    typename T::const_iterator it = container.begin(); // OK: parses as type
}
```

---

## 2. Template Argument Deduction & Implicit Conversions

**The Gotcha**: Template argument deduction does **not** perform implicit conversions (like promoting `int` to `float`).

```cpp
template <typename T>
T getMax(T a, T b) {
    return (a > b) ? a : b;
}

void demoDeductionFail() {
    // getMax(5, 5.5); // COMPILER ERROR: Conflicting types deduced for T (int vs double)
}
```

### The Solutions:
1. **Explicit template parameters**: `getMax<double>(5, 5.5)` (forces compiler to convert 5 to double, bypassing deduction).
2. **Multi-parameter templates**:
   ```cpp
   template <typename T, typename U>
   auto getMax(T a, U b) -> decltype(a > b ? a : b) {
       return (a > b) ? a : b;
   }
   ```
3. **C++20 `std::common_type`**:
   ```cpp
   #include <type_traits>
   template <typename T, typename U>
   std::common_type_t<T, U> getMax(T a, U b) {
       return (a > b) ? a : b;
   }
   ```

---

## 3. Expression Templates (Performance Optimization)

**Why Tricky**: Given a custom vector class `Vec`, evaluating expressions like `Vec D = A + B + C` naively allocates temporary vectors to store intermediate results `(A + B)`. In high-performance math or HFT codes, this memory allocation overhead is unacceptable.

**The Solution -- Expression Templates**:
- Instead of returning a new `Vec` from `operator+`, return a lightweight **proxy placeholder object** that stores references to the two operand expressions (e.g., `VecAddExpr<Vec, Vec>`).
- The actual loop addition is deferred and executed only when assigning to the destination vector. The compiler merges all additions into a single fused loop:
  `D[i] = A[i] + B[i] + C[i]`
- This is a cornerstone design of high-performance libraries like Eigen.

---

## Recap of Tricky Template Gotchas

| Gotcha | Theme |
|---|---|
| **Dependent Names** | Nested template types must be prefixed with `typename` to guide phase-1 compiler parser. |
| **Deduction Conversions** | Parameter type deduction is strict and refuses implicit casts; resolved by explicit calls or `std::common_type`. |
| **Expression Templates** | Defers vector arithmetic evaluation using proxy structures to eliminate intermediate temporary allocations. |

---

*<- [[04_Tmpl_Solutions|Solutions]] · [[../05_STL_and_Modern_Features/05_STL_Index|STL & Modern Features ->]]*
