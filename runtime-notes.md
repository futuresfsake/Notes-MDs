# Runtime

## 1. Definition

The **runtime** is the support code and services that run alongside your program while it executes. Compiled code doesn't do everything on its own. It depends on a layer of helper code that handles startup, memory, type checks, and error conditions. The runtime sits between your program and the operating system.

## 2. What the runtime does

A runtime can include several services:

- **Startup:** before `main()` runs, code sets up the stack, passes command-line arguments, and initializes global variables. In C this is a small piece of code called `_start`, which calls into the C library and then `main`.
- **Memory management:** allocating and releasing memory while the program runs.
- **Runtime checks:** verifying things during execution, such as array bounds, type compatibility, or null references.
- **Error handling:** raising and propagating exceptions or panics.
- **Library support:** functions like `printf`, string handling, or threads that the language assumes are available.

Languages differ greatly in how much of this they provide. Some put a lot of work here; others put very little.

## 3. Runtime in practice

The runtime is what makes a compiled program start and run. A simplified view of a C program's startup:

```
OS loads the executable
  └── _start (startup code from the C runtime)
        ├── sets up the stack and arguments
        ├── initializes the C library
        └── calls main()
              └── your code runs
```

When `main` returns, the runtime cleans up and returns the exit code to the operating system.

## 4. Runtime in different languages

- **C:** thin. Startup code, the standard library (`malloc`, `free`, `printf`), and the operating system. It doesn't check array bounds, track memory use, or stop you from using a freed pointer.
- **Java:** rich. The JVM provides garbage collection, bounds checks, type checks, exceptions, and just-in-time compilation.
- **Python:** rich. The interpreter checks types while the code runs, raises exceptions for out-of-range indexes, and frees memory automatically.
- **Go:** moderate. A runtime with garbage collection and a scheduler for lightweight goroutines.
- **Rust:** minimal. Ownership rules are enforced at compile time, so the runtime does little extra work.

## 5. Runtime type checking

Type checking can happen at compile time, at runtime, or both:

- **Static (compile time):** C, Java, and Rust check most types before the program runs.
- **Dynamic (runtime):** Python checks types while the code runs. Adding a string and a number raises an error only when that line executes.
- **Mixed:** Java checks types at compile time, but casts and some array operations are verified at runtime, which can throw an exception.

## 6. Trade-offs

Every runtime service has a cost. Garbage collection adds pauses and memory overhead. Runtime bounds checks add instructions. A thin runtime gives more control and predictable performance, but it leaves more safety work to the programmer. Choosing a language is partly choosing which runtime costs and guarantees you accept.

## 7. Key takeaways

- The runtime is the support code and services that run alongside your program during execution.
- It handles startup, memory management, runtime checks, error handling, and library support, to different degrees depending on the language.
- C's runtime is small; managed languages provide much more.
- Type checking can happen at compile time, at runtime, or both.

## 8. What a senior engineer would think about

When diagnosing a production problem, knowing what the runtime does helps you narrow the cause. A pause may be garbage collection, a crash may be a failed runtime check or a stack overflow, and a slow startup may be runtime initialization. Measure before you blame the runtime, and use profilers and runtime metrics to see what it's actually doing.
