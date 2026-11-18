# Intermediate Representation (IR) and the Compiler Pipeline: Study Notes

These notes summarize our discussion about how compilers turn source code into machine code, why an IR exists, and how it reduces the work of supporting many languages and many processors. They are written as a reference you can revisit.

---

## 1. The big picture

A compiler translates source code written in a high-level language into machine code that a specific processor can execute. Every compiler does this in stages:

```
Source code
    │
    ▼  Lexer: characters → tokens
Tokens
    │
    ▼  Parser: tokens → Abstract Syntax Tree (AST)
AST
    │
    ▼  Semantic analysis: check names, types, and rules
Checked AST
    │
    ▼  Lowering: AST → Intermediate Representation (IR)
IR
    │
    ▼  Optimizer: improve the IR (shared by all languages and targets)
Optimized IR
    │
    ▼  Backend: IR → machine code for one architecture
Machine code (object file)
    │
    ▼  Linker: combine object files into an executable
Executable
```

The **input** is source code. The **output** is machine code for a specific processor architecture. Note that machine code is the output, not "another language."

---

## 2. Where the IR fits

**The AST comes before the IR.** A common mistake is to think the AST is already the IR. It is not:

- The **AST** is the output of parsing. It still resembles your source code, with nodes for `if` statements, function declarations, and loops.
- The **IR** is produced by **lowering** the AST. It is simpler and more uniform, usually a sequence of operations on values with explicit control flow. It is designed to be analyzed and optimized.

Example. The C code `return a + b;` produces an AST node for "return" containing an "add" node with children `a` and `b`. Lowering turns that into IR instructions such as `%sum = add i32 %a, %b` followed by `ret i32 %sum`.

The IR is therefore the first form of the program that is not tied to any source language or processor.

---

## 3. What an IR is

An **intermediate representation** is an internal data structure a compiler uses between parsing and code generation.

Key properties:

- **Processor-neutral.** It describes operations like "add two integers," not "use register `eax`."
- **Language-neutral.** Many languages can lower into the same IR.
- **Easy to analyze.** It is typically in a form such as **SSA** (Static Single Assignment), where each value is assigned once, which makes optimizations reliable.
- **Internal.** Users rarely see it. It exists inside the compiler.

Real examples include LLVM IR (used by Clang, Rust, and Swift), GCC's GIMPLE and RTL, and MLIR (used in machine learning compilers). Some IRs are also shipped to users, such as JVM bytecode, .NET CIL, and WebAssembly. Those can be translated to machine code on the user's device.

---

## 4. Why the IR matters: the 12×12 problem

This is the central idea of the discussion.

**Without an IR:** every source language needs its own compiler for every target architecture. With 12 languages and 12 architectures:

> 12 × 12 = **144** compilers, each built and maintained separately.

**With an IR:** split the work into two layers.

- Each **language** gets one **frontend** (lexer, parser, lowering to IR): 12 frontends.
- Each **architecture** gets one **backend** (instruction selection, register allocation, code emission): 12 backends.
- One shared **optimizer** works on the IR and benefits every combination.

> 12 + 12 = **24** components instead of 144.

**What this means in practice:**

- Adding a 13th language means writing one new frontend, and it immediately works on all 12 architectures.
- Adding a 13th architecture means writing one new backend, and it immediately works for all 12 languages.
- An optimization written once for the IR improves every language and every target.

**Clarification on "a single pipeline":** the design is not literally one pipeline that handles everything. It is N frontends and M backends joined at a shared IR. The saving comes from sharing the middle, not from eliminating the per-language or per-architecture work.

### Presenting the idea

> Without an IR, each language needs a compiler for every architecture: 12 × 12 = 144 compilers. With an IR, each language gets one frontend and each architecture gets one backend, and the optimizer is shared: 12 + 12 = 24 components. Adding one language or architecture then means writing one component, not a whole row or column of compilers.

---

## 5. Processor architectures

An **architecture** is a processor design specification. Specifically, it defines the **Instruction Set Architecture (ISA)**: the machine instructions, registers, calling conventions, and memory rules that software relies on. A **processor** (CPU) is a physical chip that implements an architecture.

Examples:

| Architecture | Description | Where it's used |
|---|---|---|
| **x86-64 (AMD64)** | CISC design, variable-length instructions, strong memory ordering | Intel and AMD desktops, laptops, and servers |
| **ARM64 (AArch64)** | RISC design, fixed 4-byte instructions, weaker memory ordering | Phones, Apple Silicon Macs, AWS Graviton servers |
| **RISC-V** | Open RISC design, no licensing fees | Embedded devices, research, emerging chips |
| **PowerPC** | RISC design | Older Apple Macs, some IBM systems |
| **s390x** | IBM mainframe architecture | IBM mainframes |
| **MIPS** | RISC design | Routers, embedded devices, older consoles |
| **WebAssembly** | A *virtual* architecture | Runs inside a runtime in browsers and servers, not on a physical chip |

**CISC vs. RISC in brief:** CISC (Complex Instruction Set Computer) provides many varied instructions, some doing several steps at once, which gives compact code. RISC (Reduced Instruction Set Computer) provides fewer, simpler instructions, with memory access separated from arithmetic, which suits pipelining and power efficiency. Modern chips blur these lines internally.

### Checking your own laptop

- **Windows:** in PowerShell, run `echo $env:PROCESSOR_ARCHITECTURE` (`AMD64` = x86-64, `ARM64` = ARM).
- **macOS:** run `uname -m` (`arm64` = Apple Silicon, `x86_64` = Intel).
- **Linux:** run `uname -m` (`x86_64` = x86, `aarch64` = ARM).

The chip brand does not determine the architecture on its own. Intel and AMD make x86 chips, but Apple, Qualcomm, and others make ARM chips.

---

## 6. Architecture-specific examples

The same `add` function compiled for two architectures, from the same LLVM IR:

```llvm
define i32 @add(i32 %a, i32 %b) {
entry:
  %sum = add i32 %a, %b
  ret i32 %sum
}
```

x86-64 (System V ABI, typical output at `-O2`):

```asm
add:
    lea eax, [rdi+rsi]   ; first and second arguments in edi and esi; result in eax
    ret
```

ARM64 (AArch64, typical output):

```asm
add:
    add w0, w0, w1       ; first and second arguments in w0 and w1; result in w0
    ret
```

The IR is identical. The backends choose different instructions and registers.

---

## 7. Key concepts reference

| Term | Meaning |
|---|---|
| **Lexer** | Splits characters into tokens |
| **Parser** | Builds the AST from tokens, following grammar rules |
| **AST** | Tree representing the program's structure, still close to the source |
| **Lowering** | Converts the AST into IR |
| **IR** | Internal, processor-neutral, language-neutral representation |
| **SSA** | Static Single Assignment: each value is assigned once |
| **Optimizer** | Passes that improve the IR (dead code removal, inlining, constant folding) |
| **Frontend** | Language-specific part: source → IR |
| **Backend** | Architecture-specific part: IR → machine code |
| **ISA** | Instruction Set Architecture: the instructions, registers, and rules of a processor family |
| **ABI** | Application Binary Interface: calling conventions and data layout at the binary level |
| **Machine code** | Binary instructions the processor executes directly |
| **Assembly** | Human-readable text form of machine code |
| **Object file / Linker** | Compiled output per source file, and the step that combines them into an executable |

---

## 8. Common mistakes

1. **Treating the AST as the IR.** The AST is the parser's output; the IR comes after lowering.
2. **Calling machine code a "target language."** The target is machine code for a specific architecture.
3. **Saying the IR makes everything one pipeline.** You still write one frontend per language and one backend per architecture. The IR removes the need to write a compiler for every pair.
4. **Confusing an architecture with a brand.** x86 is an architecture family; Intel and AMD both build x86 chips.
5. **Assuming a compiled binary runs anywhere.** Native binaries target one ISA and one operating system ABI.
6. **Assuming Python compiles like C.** CPython runs bytecode in an interpreter. Tools such as Numba and PyPy generate machine code through a different path.

---

## 9. Best practices

- Explain the pipeline in stages, and identify which stage each concept belongs to.
- Quantify the benefit: N + M components instead of N × M compilers.
- Keep language-specific logic in the frontend and architecture-specific logic in the backend.
- When inspecting compiler behavior, use tools such as Compiler Explorer (godbolt.org) to see the IR and assembly side by side.
- Test on every architecture you target, especially for concurrent code, because memory ordering differs (x86 is stronger than ARM).

---

## 10. Key takeaways

- The compiler pipeline is: source → tokens → AST → IR → optimized IR → machine code.
- The IR is produced by lowering the AST, not by parsing directly.
- An IR lets a compiler support N languages and M architectures with N frontends and M backends instead of N × M compilers.
- An architecture is the processor's instruction set and rules; a processor is a chip that implements one.
- The same IR can be lowered to different machine code for each architecture, following that architecture's ABI.

---

## 11. Further reading

- *Crafting Interpreters* by Robert Nystrom (free online), for building a language from parsing through execution.
- *Engineering a Compiler* by Cooper and Torczon, for IR design and optimization.
- The LLVM Language Reference (llvm.org/docs/LangRef.html), for the exact definition of LLVM IR.
- The System V AMD64 ABI document, for x86-64 calling conventions.
- The Arm Architecture Reference Manual for AArch64.
- Compiler Explorer (godbolt.org), for experimenting with IR and assembly.
