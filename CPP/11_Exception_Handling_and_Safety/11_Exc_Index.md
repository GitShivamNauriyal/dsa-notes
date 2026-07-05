---
tags: [cpp, exceptions, safety, index]
links: ["[[../00_Home]]", "[[../10_Low_Latency_and_Quant_CPP/10_LLQ_Index]]"]
---

# Exception Handling & Safety -- Index

*<- [[../10_Low_Latency_and_Quant_CPP/10_LLQ_Index|Low-Latency & Quant C++]] · [[11_Exc_Exceptions_vs_Error_Codes\|01 · Exceptions vs Error Codes ->]]*

---

| File | Topics |
|------|--------|
| [[11_Exc_Exceptions_vs_Error_Codes\|01 · Exceptions vs Error Codes]] | Performance tradeoffs, table comparison, `std::expected` (C++23) |
| [[11_Exc_Noexcept_and_Specifications\|02 · Noexcept & Specifications]] | `noexcept` specifiers, compile-time checks (`noexcept` operator), vector optimization |
| [[11_Exc_Exception_Safety_Guarantees\|03 · Exception Safety Guarantees]] | Basic, Strong, and Nothrow/No-fail exception safety guarantees, copy-and-swap |
| [[11_Exc_Stack_Unwinding_and_RAII\|04 · Stack Unwinding & RAII]] | Destructor cleanups, stack unwinding timelines, resource leaks prevention |
| [[11_Exc_Problems\|05 · Concept Checks & Exception Problems]] | Class design and safety guarantees exercises |
| [[11_Exc_Solutions\|06 · Worked Solutions]] | Complete solutions to the exception safety challenges |
| [[11_Exc_Tricky\|07 · Tricky Exception Gotchas]] | Destructors throwing during unwinding, move reallocations, thread boundaries (`std::exception_ptr`) |

---

*<- [[../10_Low_Latency_and_Quant_CPP/10_LLQ_Index|Low-Latency & Quant C++]] · [[11_Exc_Exceptions_vs_Error_Codes\|01 · Exceptions vs Error Codes ->]]*
