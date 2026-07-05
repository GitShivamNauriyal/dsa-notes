---
tags: [cpp, core, solutions]
links: ["[[02_Core_Index]]", "[[02_Core_Problems]]", "[[02_Core_Tricky]]"]
---

# Core Language Mechanics -- Solutions

*<- [[02_Core_Problems|Problems]] · [[02_Core_Tricky|Tricky Mechanics ->]]*

---

## Tier 1 -- Concept Checks

### Solution 1.1: Value Categories
1. `x`: **lvalue**. It has a name and has identity.
2. `lref`: **lvalue**. Even though it is a reference, the expression referring to it has a name.
3. `rref`: **lvalue**. An rvalue reference variable itself is an lvalue because it has a name.
4. `std::move(x)`: **xvalue** (eXpiring value). It has identity but is cast to an rvalue reference, signifying it can be safely moved from.
5. `42`: **prvalue** (pure rvalue). It is a literal and has no identity.
6. `x + 5`: **prvalue**. The temporary result of an arithmetic expression has no address/identity.

---

### Solution 1.2: Linkage and Storage Duration
1. `App::count`: **External linkage**, **Static storage duration** (lasts the entire program runtime).
2. `static int counter`: **No linkage** (only visible inside the function block scope), **Static storage duration** (memory persists across function calls).
3. `static int internalVal`: **Internal linkage** (visible only inside its translation unit), **Static storage duration**.
4. `int x` (parameter): **No linkage**, **Automatic storage duration** (allocated on the stack and freed when the function returns).

---

## Tier 2 -- Implementation

### Solution 2.1: Constexpr Factorial

```cpp
#include <iostream>

constexpr long long factorial(int n) {
    long long res = 1;
    for (int i = 2; i <= n; i++) {
        res *= i;
    }
    return res;
}

int main() {
    // 1. Forced compile-time evaluation
    constexpr long long val = factorial(5); // Computed at compile-time
    std::cout << val << "\n";

    // 2. Runtime evaluation
    int n = 5;
    long long val2 = factorial(n);          // Computed at runtime
    std::cout << val2 << "\n";
    return 0;
}
```

---

## Tier 3 -- Interview-Level

### Solution 3.1: Strict Aliasing Violation Debugging

1. **Why it violates strict aliasing**:
   - `int` and `float` are incompatible types under the strict aliasing rule.
   - By casting `float*` to `int*` and dereferencing it, you are telling the compiler to treat float memory as an integer.
   - The compiler is allowed to assume that changes made through an `int*` cannot affect a `float`. Therefore, it may reorder writes or optimize away reads based on this assumption, leading to corrupted data or undefined runtime crashes.

2. **Safe Alternatives**:

```cpp
#include <cstring>
#include <bit>
#include <cstdint>
#include <iostream>

// C++11 Safe Implementation using std::memcpy
uint32_t inspectBitsCpp11(float f) {
    uint32_t bits;
    // std::memcpy accesses memory via char/byte arrays, which is a defined exception to strict aliasing
    std::memcpy(&bits, &f, sizeof(f));
    return bits;
}

// C++20 Safe Implementation using std::bit_cast
uint32_t inspectBitsCpp20(float f) {
    return std::bit_cast<uint32_t>(f); // Done at compile-time if input is constexpr!
}
```

---

### Solution 3.2: Compile-Time Verification of Array Size

We can use `static_assert` paired with a template matching function to verify array bounds at compile-time.

```cpp
// C++11 compile-time size checking helper
template <typename T, std::size_t N>
constexpr std::size_t getArraySize(const T (&)[N]) {
    return N;
}

#define CHECK_ARRAY_SIZE(arr, expectedSize) \
    static_assert(getArraySize(arr) == expectedSize, "Array size does not match expected size!")

void test() {
    int myArr[] = {1, 2, 3};
    CHECK_ARRAY_SIZE(myArr, 3); // Passes compilation
    // CHECK_ARRAY_SIZE(myArr, 4); // Fails compilation!
}
```

---

## Tier 4 -- Systems / Placement-Hard

### Solution 4.1: Custom `bit_cast` for C++11 Floor

To implement `bit_cast` in C++11 safely, we must use:
1. `static_assert` to check size equivalence.
2. `std::is_trivially_copyable` to ensure bitwise copy safety.
3. `std::memcpy` to copy the bits without violating strict aliasing.

```cpp
#include <type_traits>
#include <cstring>

template <typename Dest, typename Source>
Dest custom_bit_cast(const Source& src) {
    // Compile-time checks
    static_assert(sizeof(Dest) == sizeof(Source), 
                  "Source and Destination types must have identical sizes!");
    
    static_assert(std::is_trivially_copyable<Source>::value,
                  "Source type must be trivially copyable!");
    
    static_assert(std::is_trivially_copyable<Dest>::value,
                  "Destination type must be trivially copyable!");

    Dest dest;
    std::memcpy(&dest, &src, sizeof(Source)); // Safe bit-level copy
    return dest;
}
```

---

*<- [[02_Core_Problems|Problems]] · [[02_Core_Tricky|Tricky Mechanics ->]]*
