# Intermediate Representation (IR)

## 1. Definition

An **Intermediate Representation (IR)** is a data structure a compiler uses internally, between reading the source code and producing the final output. It's designed to be simpler and more uniform than any real programming language, and it's usually **machine-independent**: it doesn't describe x86 or ARM registers and instructions.

The IR is not the final output. It's the compiler's working form, the stage where the program is easiest to analyze and transform. The IR is the data; the optimizations and analyses that run over it are algorithms.

## 2. Where the IR lives

The IR lives **inside the compiler**. It's never written to a file for the user, sent anywhere, or executed by the CPU. It exists only while the compiler is working.

The full journey of a C program looks like this:

```
┌──────────────┐     ┌─────────────────────────────────────────────────┐     ┌───────────────┐     ┌─────────┐
│  C source    │ ──▶ │                    COMPILER                     │ ──▶ │ Machine code  │ ──▶ │   CPU   │
│  (you write) │     │                                                 │     │ (the output)  │     │ (runs   │
│              │     │  front end ──▶ IR ──▶ optimizer ──▶ IR ──▶ back end  │     │               │     │  it)    │
└──────────────┘     │                 (IR changes inside here)        │     └───────────────┘     └─────────┘
                     └─────────────────────────────────────────────────┘
```

In order:

1. **C source:** the code you write. It's the input.
2. **Compiler:** reads the source, builds the IR, improves the IR, and generates machine code. The IR is created and changed entirely inside this box.
3. **Machine code:** the compiler's output, the target. It's usually stored as a binary file.
4. **CPU:** executes the machine code. It never sees the source or the IR.

Inside the compiler, the stages are:

- **Front end** (lexing, parsing, semantic analysis): reads the C source and turns it into the IR.
- **Optimizer:** rewrites the IR to make the program faster or smaller, while keeping its meaning the same.
- **Back end:** translates the final IR into machine code for a specific target.

## 3. Source languages and targets

Two terms are easy to mix up:

- **Source language:** the language you write in. If you write C and compile it with GCC or Clang, C is the source language. It's the input.
- **Target:** the output the compiler produces. It's not the language you typed.

When you compile C for an x86 laptop, the target is x86 machine code, and the final output is a binary the CPU executes directly. "Target language" is a more general term, because a compiler can emit several kinds of output:

- **Machine code** for a CPU, such as x86, ARM, or RISC-V.
- **Bytecode** for a virtual machine, such as Java bytecode or WebAssembly.
- **Another high-level language**, which is what a transpiler does. TypeScript compiling to JavaScript is an example. In that case, the target really is a language.

So a C program's target is machine code unless you use a transpiler.

## 4. Purpose

The IR makes the program easier for the compiler to reason about. Source languages have many constructs, such as `for` loops, `for`-each loops, operator overloading, and syntactic sugar. The IR reduces them to a smaller set of operations, so each optimization or analysis handles fewer cases.

It also gives the compiler a stable middle layer to transform. New optimizations can be added later and run on the IR without touching the front ends or back ends.

### What "transformations" means

A transformation is any step that rewrites the program while preserving its meaning. Common kinds include:

- **Optimizations**, which make the program faster or smaller:
  - *Constant folding:* `x = 2 * 3` becomes `x = 6`.
  - *Dead-code elimination:* removing code whose result is never used.
  - *Inlining:* replacing a function call with the function's body.
  - *Loop-invariant code motion:* moving calculations that don't change inside a loop to before the loop.
- **Analyses**, which examine the program without changing it, but inform transformations:
  - Finding which variables are live at a given point.
  - Checking for uninitialized reads.
- **Lowering**, which converts high-level constructs into simpler ones. A `for`-each loop might become an index-based loop, and a method call might become a function call with a hidden pointer argument.
- **Instrumentation**, which adds extra code, such as bounds checks or profiling counters.

The word "future" points at extensibility. The IR is designed so new transformations can be added later without rewriting the front ends or back ends. A new optimization written against the IR works for every source language and every target.

### The N × M problem

Suppose you have **N** source languages (C, C++, Rust, Swift) and **M** targets (x86, ARM, RISC-V, WebAssembly).

- **Without an IR:** each language needs its own compiler for each target. That's N × M = 4 × 4 = **16** compilers.
- **With an IR:** each language has one front end that translates into the IR (N components), and each target has one back end that translates from the IR (M components). That's N + M = 4 + 4 = **8** components.

Optimizations also get written once, against the IR, and they benefit every language and every target. The gap widens as N and M grow: 10 × 10 needs 100 compilers without an IR and only 20 with one.

## 5. Properties of a good IR

- **Simple and uniform:** a small set of operations with consistent rules.
- **Well defined:** the meaning of every construct is precise, so optimizations can be proven safe.
- **Easy to analyze:** it supports control-flow and data-flow analysis.
- **Target-neutral:** it doesn't commit to any particular CPU's features.

## 6. Common forms of IR

- **Abstract syntax trees (AST):** closest to the source; used by many front ends, but less convenient for optimization.
- **Three-address code:** each instruction has at most one operator and up to three operands, such as `t1 = a + b`.
- **Control-flow graphs (CFG):** basic blocks connected by jumps, which make loops and branches explicit.
- **Static Single Assignment (SSA) form:** every variable is assigned exactly once, which makes many optimizations simpler.

## 7. Real-world examples

- **LLVM IR:** the best-known shared IR. Clang (C/C++), Rust, and Swift all target it, and LLVM's back ends generate code for x86, ARM, RISC-V, and WebAssembly.
- **GCC GIMPLE and RTL:** GCC's internal IRs, used for optimization at different levels.
- **Go SSA:** Go's compiler uses an SSA-based IR internally for optimization, even though it targets only a few platforms.
- **Rust MIR:** Rust's mid-level IR, used for borrow checking and optimization.
- **Java bytecode and WebAssembly:** stack-based intermediate formats. The JVM and browsers translate them to machine code at load or run time. These are shipped outputs, so they're a different case from an internal IR.

## 8. Is an IR always needed?

No. An IR is a design choice.

- **Single language and single target:** N × M = 1, so a shared IR brings no reuse. A direct pipeline is valid.
- **Transpilers** (such as TypeScript to JavaScript) emit source code, so they don't need a machine-level representation.
- **Tree-walking interpreters** execute the AST directly.
- **Single-compiler IRs** such as Go's SSA or Rust's MIR are used for optimization and maintainability, even without sharing.

An IR pays off when you have many languages or targets, want optimizations written once, or want third parties to build on your compiler.

## 9. Trade-offs

- **Design effort:** a stable, uniform representation is hard to define.
- **Compile-time overhead:** each extra stage takes time, though usually a small share.
- **Debugging complexity:** bugs can hide in translation layers.
- **Lost information:** lowering to a simpler form can discard details the source language needed, such as precise error locations.

## 10. Key takeaways

- The IR is an internal data structure inside the compiler. It's never sent out, and the CPU never sees it.
- The flow is: C source → compiler (builds and improves the IR) → machine code → CPU runs it.
- The source language is the input you write; the target is the output, most often machine code.
- Optimizations, analyses, and lowering are algorithms that transform the IR while preserving meaning.
- A shared IR solves the N × M problem, reducing N × M compilers to N + M components.
- Using an IR is a design choice. A shared IR pays off across many languages and targets; a private IR can pay off for maintainability alone.

## 11. What a senior engineer would think about

Don't add an IR speculatively. Start with the simplest pipeline that meets your needs, and introduce an IR when you feel the pain it solves, such as duplicated optimizations or the N × M explosion. A small, well-defined IR inside a single compiler is often valuable even if it's never shared. Design the IR carefully, because it becomes a contract once other tools depend on it, and changing it is expensive. That's why LLVM IR has been so influential and so hard to evolve.
