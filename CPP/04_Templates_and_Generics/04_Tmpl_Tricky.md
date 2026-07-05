---
tags: [cpp, templates, tricky, dependent-templates, dependent-names, expression-templates]
links: ["[[04_Tmpl_Index]]", "[[04_Tmpl_Solutions]]", "[[../05_STL_and_Modern_Features/05_STL_Index]]"]
---

# Templates & Generics -- Tricky & Higher-Order

*<- [[04_Tmpl_Solutions|Solutions]] · [[../05_STL_and_Modern_Features/05_STL_Index|STL & Modern Features ->]]*

---

## 1. Dependent Names & `typename`

When writing a template, if you access a nested name (such as a type definition or an inner class) that depends on a template parameter, the compiler assumes it is a **static variable** during Phase 1 parsing.

- **The Consequence**: A statement like `T::const_iterator * ptr;` is parsed as a multiplication expression (`T::const_iterator` multiplied by `ptr`) instead of a pointer declaration!
- **The Fix**: Prepend **`typename`** to instruct the parser that the dependent name is a type.

```cpp
#include <vector>

template <typename T>
void iterateContainer(const T& container) {
    // typename forces phase-1 parser to treat T::const_iterator as a type
    typename T::const_iterator it = container.begin(); 
}
```

---

## 2. Dependent Template Members & the `template` Keyword

Similar to the `typename` issue, if you call a member template function (or access a nested template member) of a class that depends on a template parameter, the compiler parses the `<` symbol as a **less-than comparison operator** during Phase 1 parsing.

```cpp
template <typename T>
void execute(T& obj) {
    // Intention: Call obj.get<int>()
    // Reality: COMPILER ERROR! Parser treats this as: (obj.get < int) > ()
    // obj.get<int>(); 
}
```

### The Fix:
You must explicitly insert the **`template`** keyword before the member name to instruct the parser that `<` starts a template argument list.

```cpp
template <typename T>
void execute(T& obj) {
    // Tells the compiler obj.get is a template member
    obj.template get<int>(); // OK
}
```

---

## 3. Template Argument Deduction & Implicit Conversions

Template argument deduction is **strict** and does not perform implicit promotions (e.g. promoting `int` to `double` or converting a subclass pointer to a base class pointer).

```cpp
template <typename T>
T addValues(T a, T b) { return a + b; }

void demoDeductionFail() {
    // addValues(10, 20.5); // COMPILER ERROR: conflicting types deduced for T (int vs double)
}
```

### The Solutions:
1. **Explicit Call**: `addValues<double>(10, 20.5)` (converts 10 to double, skipping deduction).
2. **`std::common_type`** (C++11): Deduce a common promoted return type.
   ```cpp
   #include <type_traits>
   template <typename T, typename U>
   std::common_type_t<T, U> addValues(T a, U b) { return a + b; }
   ```

---

## 4. Expression Templates (Performance Optimization)

Evaluating expressions like `Vec D = A + B + C` naively allocates intermediate temporary objects to store `(A + B)`, leading to high memory overhead.

**Expression Templates** solve this by:
- Overloading `operator+` to return a lightweight **proxy placeholder object** (`VecAddExpr<A, B>`) storing references to the operands, rather than a new vector.
- The computation is deferred and executed only when assigning to the destination vector, allowing the compiler to fuse the entire expression into a single loop:
  `D[i] = A[i] + B[i] + C[i]`
- This is a critical pattern used in high-performance linear algebra libraries like Eigen to achieve zero-overhead vector arithmetic.

---

## Recap of Tricky Template Gotchas

| Gotcha | Theme |
|---|---|
| **Dependent Names** | Nested dependent type names must be prefixed with `typename` to guide phase-1 parser. |
| **Dependent Templates** | Nested dependent template member calls must insert the `template` keyword before the member name to prevent `<` being parsed as a comparison. |
| **Deduction Conversions** | Parameter type deduction is strict and refuses implicit casts; resolved by explicit calls or `std::common_type`. |
| **Expression Templates** | Defers vector arithmetic evaluation using proxy structures to eliminate intermediate temporary allocations. |

---

*<- [[04_Tmpl_Solutions|Solutions]] · [[../05_STL_and_Modern_Features/05_STL_Index|STL & Modern Features ->]]*
