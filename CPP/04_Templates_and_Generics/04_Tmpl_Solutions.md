---
tags: [cpp, templates, solutions]
links: ["[[04_Tmpl_Index]]", "[[04_Tmpl_Problems]]", "[[04_Tmpl_Tricky]]"]
---

# Templates & Generics -- Solutions

*<- [[04_Tmpl_Problems|Problems]] · [[04_Tmpl_Tricky|Tricky Template Gotchas ->]]*

---

## Tier 1 -- Concept Checks

### Solution 1.1: Function Template Specialization
- **Compilation Outcome**: **Fails compilation!**
- **Explanation**: 
  - The code attempts to **partially specialize** the function template `printVal` by binding the second parameter `U` to `int` while keeping `T` generic.
  - C++ standard does not allow partial specialization of function templates.
  - **Workaround Fix**: Use function overloading:
    ```cpp
    template <typename T>
    void printVal(T a, int b) { // Overload, not partial specialization
        std::cout << "Overload A\n";
    }
    ```

---

### Solution 1.2: Variadic Template Instantiation count
1. `getArgsCount(1, 2.5, "Hello")` returns **3** (evaluated at compile-time by `sizeof...` operator).
2. The compiler generates **exactly 1** distinct function instantiation for this specific type signature: `int getArgsCount<int, double, const char*>(int, double, const char*)`.
   - If `getArgsCount` is called elsewhere with different types (e.g. `getArgsCount(1, 2)`), a new distinct function is instantiated.

---

## Tier 2 -- Implementation

### Solution 2.1: Print Pack Fold Expression

To print parameters with a comma between them but not after the last, we can split the pack: print the first element on its own, and then print subsequent elements prefixed with a comma using a fold expression.

```cpp
#include <iostream>

// Helper wrapper to print with preceding comma
template <typename T>
void printWithComma(const T& val) {
    std::cout << ", " << val;
}

// Primary variadic template
template <typename First, typename... Rest>
void printPack(const First& first, const Rest&... rest) {
    std::cout << first; // Print the first element
    
    // Fold expression: calls printWithComma for every element in 'rest'
    (printWithComma(rest), ...);
    std::cout << "\n";
}

int main() {
    printPack(1, "two", 3.14); // Outputs: 1, two, 3.14
    return 0;
}
```

---

## Tier 3 -- Interview-Level

### Solution 3.1: Re-implementing Type Traits

We use template specialization. The primary template handles non-pointers. We partially specialize for raw pointers `T*` and const pointers `const T*`.

```cpp
// 1. Primary template
template <typename T>
struct is_pointer_type {
    static constexpr bool value = false;
};

// 2. Partial specialization for all pointer types
template <typename T>
struct is_pointer_type<T*> {
    static constexpr bool value = true;
};

// 3. Partial specialization for const pointer types
template <typename T>
struct is_pointer_type<T* const> {
    static constexpr bool value = true;
};
```

---

### Solution 3.2: SFINAE Member Checker

We use C++11 `std::true_type` and `std::false_type` helpers. We attempt to resolve a helper function using `decltype` on the member call expression.

```cpp
#include <type_traits>
#include <iostream>

template <typename T>
class has_serialize_member {
    // Helper signatures
    template <typename C>
    static auto test(int) -> decltype(std::declval<C>().serialize(), std::true_type{});

    template <typename C>
    static std::false_type test(...); // fallback catch-all

public:
    // If substitution of test(0) succeeds using the first overload, type resolves to std::true_type
    static constexpr bool value = decltype(test<T>(0))::value;
};

// Verification structures
struct Printable { void serialize() {} };
struct Empty {};

void verify() {
    std::cout << has_serialize_member<Printable>::value << "\n"; // Prints 1 (true)
    std::cout << has_serialize_member<Empty>::value << "\n";     // Prints 0 (false)
}
```

---

## Tier 4 -- Systems / Placement-Hard

### Solution 4.1: Compile-Time Typelist and Length Checker

```cpp
#include <type_traits>

// 1. Typelist structure
template <typename... Types>
struct TypeList {};

// 2. Length Checker
template <typename TL>
struct TypeListLength;

template <typename... Types>
struct TypeListLength<TypeList<Types...>> {
    static constexpr int value = sizeof...(Types);
};

// 3. TypeAt Index Checker
template <typename TL, int Index>
struct TypeAt;

// Base case: index is 0, extract the first type
template <typename Head, typename... Tail>
struct TypeAt<TypeList<Head, Tail...>, 0> {
    using type = Head;
};

// Recursive case: index > 0, decrement index and recurse on the tail
template <typename Head, typename... Tail, int Index>
struct TypeAt<TypeList<Head, Tail...>, Index> {
    using type = typename TypeAt<TypeList<Tail...>, Index - 1>::type;
};

// Verification:
void verifyTypeList() {
    using MyList = TypeList<int, double, char>;

    static_assert(TypeListLength<MyList>::value == 3, "Length must be 3");
    
    // TypeAt index 1 is double
    static_assert(std::is_same<typename TypeAt<MyList, 1>::type, double>::value, "Index 1 is double");
}
```

---

*<- [[04_Tmpl_Problems|Problems]] · [[04_Tmpl_Tricky|Tricky Template Gotchas ->]]*
