# How C Differs from Other Languages

## 1. Summary

C is often called "low level." That label means C exposes the machine model directly. Manual memory management is the most visible result, but it's not the only one. This document covers memory in C, how other languages handle memory differently, and the other ways C differs.

## 2. Memory in C

C has three main places where memory can live:

- **Stack:** local variables, such as `int x = 5;` inside a function. They're created when the function is called and removed automatically when it returns. You never call `free` on them.
- **Static/global:** variables that exist for the whole program. They're also not freed manually.
- **Heap:** memory you request at runtime with `malloc`, `calloc`, or `realloc`. This is the only kind you must release with `free`.

```c
int *numbers = malloc(10 * sizeof(int)); // heap: you asked for it
numbers[0] = 42;
free(numbers);                            // you must give it back
```

Every heap allocation needs a matching `free`, and you must not use the memory after freeing it. Forgetting `free` causes a **memory leak**. Freeing twice or using memory after freeing causes **undefined behavior**, which may crash, corrupt data, or appear to work until it doesn't.

## 3. How other languages handle memory

| Language | How memory is handled | What the programmer does |
|---|---|---|
| **C** | Manual: `malloc` and `free` | Decides every allocation's lifetime and frees it |
| **Java, Go, Python, C#** | Garbage collection: the runtime finds unreachable objects and frees them | Doesn't free memory, but can still leak by holding references |
| **Rust** | Ownership: the compiler tracks who owns each value and frees it when the owner goes out of scope | Follows ownership rules; the compiler rejects use-after-free |
| **C++** | RAII and smart pointers: cleanup is tied to object lifetime; raw `new` and `delete` still exist | Uses smart pointers like `std::unique_ptr`; manual `delete` is discouraged |

The difference isn't that other languages have no memory management. It's who handles it: you, a garbage collector at runtime, or the compiler's rules.

## 4. Other ways C differs

- **Pointers are addresses.** You can do arithmetic on them, and they can point almost anywhere.
- **Data layout is visible.** You control struct layout, padding, and alignment.
- **Arrays don't know their length.** Nothing stops you from reading or writing past the end.
- **Types are mostly erased at runtime.** The compiler checks types, but the running program doesn't carry type information the way Java or Python objects do.
- **Few runtime safety nets.** There are no built-in bounds checks, no exceptions for bad memory access, and no automatic cleanup.
- **Minimal standard library.** C provides fewer high-level features, so programs build more themselves or rely on external libraries.

Compare Python, where the runtime checks types, raises an exception for an out-of-range index, and handles memory automatically. The price is overhead and less direct control over the hardware.

## 5. Trade-offs

Low-level control gives predictable performance, small memory footprints, and direct hardware access. That's why operating systems, embedded firmware, and database engines are often written in C or C++.

The costs are real. Manual memory management is a common source of security vulnerabilities, such as buffer overflows, use-after-free bugs, and double frees. Managed languages trade some control and speed for safety and productivity. Garbage collection adds pauses and memory overhead, and runtime checks add instructions.

## 6. Key takeaways

- In C, you manage heap memory with `malloc` and `free`. Stack and global memory are handled automatically.
- Only heap allocations need a matching `free`.
- Other languages automate memory handling through garbage collection, compile-time ownership rules, or RAII.
- "Low level" in C means pointers are addresses, data layout is visible, and there are few runtime safety checks.

## 7. What a senior engineer would think about

Decide who owns each allocation before you write the `malloc` call, and make sure the code frees memory on every exit path. Use tools like AddressSanitizer, Valgrind, and static analyzers to catch mistakes the runtime won't. In production, many teams choose C or C++ where performance and control matter most, and managed or ownership-based languages where safety and development speed matter more.
