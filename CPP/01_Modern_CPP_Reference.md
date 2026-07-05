---
tags: [cpp, modern-cpp, reference]
links: ["[[00_Home]]"]
---

# Modern C++ Reference Cheat Sheet

*<- [[00_Home|Master Roadmap]] · [[02_Core_Language_Mechanics/02_Core_Index|Core Language Mechanics ->]]*

---

## C++11 Standard

| Feature | Description | Target Chapter / Link |
|---|---|---|
| `auto` | Automatic type deduction at compile time. | [[02_Core_Language_Mechanics/02_Core_Index\|Ch 2: Core]] |
| Lambdas | Inline anonymous functions. | [[05_STL_and_Modern_Features/05_STL_Index\|Ch 5: STL & Lambdas]] |
| Rvalue References & Move Semantics | Avoids deep copying by stealing resources. | [[06_Memory_and_Smart_Pointers/06_Mem_Index\|Ch 6: Smart Pointers & Move]] |
| Smart Pointers | `std::unique_ptr`, `std::shared_ptr`, `std::weak_ptr`. | [[06_Memory_and_Smart_Pointers/06_Mem_Index\|Ch 6: Memory & Smart Pointers]] |
| `nullptr` | Type-safe null pointer literal. | [[02_Core_Language_Mechanics/02_Core_Index\|Ch 2: Core]] |
| Range-based `for` loops | Loop over sequences easily: `for (auto& x : container)`. | [[05_STL_and_Modern_Features/05_STL_Index\|Ch 5: STL]] |
| Uniform Initialization | Brace initialization `{}` preventing narrowing conversions. | [[02_Core_Language_Mechanics/02_Core_Index\|Ch 2: Core]] |
| Variadic Templates | Templates accepting variable number of arguments. | [[04_Templates_and_Generics/04_Tmpl_Index\|Ch 4: Templates & Generics]] |
| `override` / `final` | Compile-time checks for virtual method overrides. | [[03_OOP_in_CPP/03_OOP_Index\|Ch 3: OOP in C++]] |
| `enum class` | Strongly-typed and scoped enumerations. | [[02_Core_Language_Mechanics/02_Core_Index\|Ch 2: Core]] |
| Concurrency primitives | `std::thread`, `std::mutex`, `std::atomic<T>`. | [[07_Concurrency/07_Conc_Index\|Ch 7: Concurrency]] |

---

## C++14 Standard

| Feature | Description | Target Chapter / Link |
|---|---|---|
| Generic Lambdas | Lambdas accepting `auto` parameters. | [[05_STL_and_Modern_Features/05_STL_Index\|Ch 5: STL & Lambdas]] |
| Lambda Init-capture | Capture expressions by moving them into the closure. | [[05_STL_and_Modern_Features/05_STL_Index\|Ch 5: STL & Lambdas]] |
| `decltype(auto)` | Deduce return type keeping reference/const qualifiers. | [[02_Core_Language_Mechanics/02_Core_Index\|Ch 2: Core]] |
| Relaxed `constexpr` | Multiple statements, loops, and conditions inside constexpr. | [[02_Core_Language_Mechanics/02_Core_Index\|Ch 2: Core]] |
| `std::make_unique` | Exception-safe unique pointer allocation factory. | [[06_Memory_and_Smart_Pointers/06_Mem_Index\|Ch 6: Smart Pointers]] |

---

## C++17 Standard

| Feature | Description | Target Chapter / Link |
|---|---|---|
| Structured Bindings | Unpack tuples, pairs, or structs: `auto [x, y] = pair;`. | [[05_STL_and_Modern_Features/05_STL_Index\|Ch 5: STL]] |
| `if constexpr` | Compile-time branch evaluation based on templates. | [[04_Templates_and_Generics/04_Tmpl_Index\|Ch 4: Templates & Generics]] |
| Variant types | `std::optional`, `std::variant`, `std::any`. | [[05_STL_and_Modern_Features/05_STL_Index\|Ch 5: STL]] |
| Fold Expressions | Parameter pack expansion with binary/unary operators. | [[04_Templates_and_Generics/04_Tmpl_Index\|Ch 4: Templates]] |
| CTAD | Class Template Argument Deduction. | [[04_Templates_and_Generics/04_Tmpl_Index\|Ch 4: Templates]] |
| `std::string_view` | Non-owning lightweight view of a string. | [[09_Performance_and_Optimization/09_Perf_Index\|Ch 9: Performance]] |

---

## C++20 Standard

| Feature | Description | Target Chapter / Link |
|---|---|---|
| Concepts | Constraints on template parameters evaluated at compile-time. | [[04_Templates_and_Generics/04_Tmpl_Index\|Ch 4: Templates & Generics]] |
| Ranges & Views | Composability of algorithms with lazy evaluation. | [[05_STL_and_Modern_Features/05_STL_Index\|Ch 5: STL & Ranges]] |
| `std::span` | Bounds-safe view over contiguous memory. | [[06_Memory_and_Smart_Pointers/06_Mem_Index\|Ch 6: Memory]] |
| Spaceship Operator `<=>` | Three-way comparison generator. | [[03_OOP_in_CPP/03_OOP_Index\|Ch 3: OOP in C++]] |
| `std::jthread` | Cooperatively interruptible thread with auto-join. | [[07_Concurrency/07_Conc_Index\|Ch 7: Concurrency]] |
| `std::atomic_ref` | Perform atomic operations on non-atomic variables. | [[07_Concurrency/07_Conc_Index\|Ch 7: Concurrency]] |
| Semaphores & Latches | `std::counting_semaphore`, `std::latch`, `std::barrier`. | [[07_Concurrency/07_Conc_Index\|Ch 7: Concurrency]] |
| Consteval / Constinit | `consteval` guarantees compile-time; `constinit` checks init. | [[02_Core_Language_Mechanics/02_Core_Index\|Ch 2: Core]] |

---

## C++23 Standard (Newest)

| Feature | Description | Target Chapter / Link |
|---|---|---|
| `std::expected` | Monadic error handling alternative to exceptions. | [[11_Exception_Handling_and_Safety/11_Exc_Index\|Ch 11: Exceptions]] |
| `std::mdspan` | Multi-dimensional span view over contiguous data. | [[10_Low_Latency_and_Quant_CPP/10_LLQ_Index\|Ch 10: Low-Latency]] |
| Deducing `this` | Explicit object parameter to simplify const/ref overloads. | [[03_OOP_in_CPP/03_OOP_Index\|Ch 3: OOP in C++]] |
| `if consteval` | Branch execution checking if evaluated at compile-time. | [[02_Core_Language_Mechanics/02_Core_Index\|Ch 2: Core]] |
| `std::print` / `std::println` | Type-safe formatted printing utility. | [[05_STL_and_Modern_Features/05_STL_Index\|Ch 5: STL]] |
| `std::views::enumerate` | Zip counter index with range element. | [[05_STL_and_Modern_Features/05_STL_Index\|Ch 5: STL]] |
| `std::views::zip` | Zip multiple ranges into tuple-like range elements. | [[05_STL_and_Modern_Features/05_STL_Index\|Ch 5: STL]] |

---

*<- [[00_Home|Master Roadmap]] · [[02_Core_Language_Mechanics/02_Core_Index|Core Language Mechanics ->]]*
