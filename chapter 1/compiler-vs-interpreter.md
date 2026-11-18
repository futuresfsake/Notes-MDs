# Compiler vs Interpreter

## The short answer

A **compiler** translates a whole program into another form before it runs, such as machine code, bytecode, or another language. The program then runs from that output.

An **interpreter** reads the program and carries out its instructions, either directly or by running an internal form it produced just before running.

Both read and understand the source code. The difference is what happens between reading and running. Compilers and interpreters sit at two ends of a spectrum, and many real systems combine them.

## Key differences

| Aspect | Compiler | Interpreter |
|---|---|---|
| When translation happens | Before the program runs | While the program runs (or just before, line by line or as a whole) |
| What gets run | A separate output: machine code, bytecode, or other source | The source, or an internal form produced by the interpreter |
| Startup time | Usually slower, because the whole program is translated first | Usually faster to start, because it begins executing right away |
| Execution speed | Usually faster, because work is done ahead of time | Usually slower, because analysis and dispatch happen while running |
| Error detection | Many errors (syntax, some type errors) found before running | Syntax errors often found as code is reached, so later code may not be checked yet |
| Output artifact | Produces a file, such as an executable, `.class`, or `.pyc` | Usually produces no standalone file (some cache internal bytecode) |
| Portability | The output is tied to a target CPU or VM, so you may need to rebuild | The same source runs anywhere the interpreter runs |
| Typical use | Production software, performance-critical systems | Scripting, prototyping, automation, configuration, interactive use |

## Important nuances

- **"Compiled" or "interpreted" usually describes an implementation, not a language.** Python is often called interpreted, but CPython compiles source to bytecode first. C is usually compiled, but interpreters for C exist.
- **Most modern systems are hybrids.** CPython compiles to bytecode and interprets it. The JVM interprets bytecode and JIT-compiles hot code. V8 uses a bytecode interpreter and then a JIT compiler.
- **A compiled program can still use an interpreter.** A bash script that runs `gcc` uses the bash interpreter for the glue, while `gcc` is a compiler.

## Examples

### Compilers

| Tool or language | Input | Output | Notes |
|---|---|---|---|
| GCC | C, C++ | Machine code (executable or object file) | Ahead-of-time compilation |
| Clang | C, C++, Objective-C | Machine code via LLVM IR | Uses an intermediate representation |
| rustc | Rust | Machine code | Ahead-of-time compilation |
| Go compiler (`go build`) | Go | Machine code | Produces a standalone binary |
| `javac` | Java | Bytecode (`.class` files) | Bytecode then runs on the JVM |
| Cython | Cython source (Python-like) | C code | A transpiler; GCC or Clang then produces machine code |
| `tsc` | TypeScript | JavaScript | A transpiler |
| `lua` compile step (`luac`) | Lua | Bytecode | Lua also runs source directly through the same machinery |

### Interpreters

| Tool or language | Runs | Notes |
|---|---|---|
| CPython | Python source (through bytecode) | Hybrid: compiles internally, then interprets |
| bash, zsh | Shell scripts | Reads and executes commands line by line |
| PowerShell | `.ps1` scripts | Shell and scripting language |
| Node.js (V8) | JavaScript | Bytecode interpreter (Ignition) plus JIT compilation |
| Ruby (YARV) | Ruby source | Compiles to bytecode, then runs it in a VM |
| PHP (Zend Engine) | PHP source | Compiles to opcodes, then executes them |
| Lua (`lua` command) | Lua source | Interprets bytecode in a virtual machine |
| Python REPL | Typed-in Python | Interactive interpreter |
| SQL engines | Query expressions and plans | Interpret query operators |
| Spreadsheets | Formulas | Evaluate formulas on change |

### Hybrids (both)

| System | How it combines both |
|---|---|
| CPython | Compiles source to bytecode, then interprets it |
| JVM (HotSpot) | Interprets bytecode, then JIT-compiles hot methods to machine code |
| V8 (JavaScript) | Bytecode interpreter, then TurboFan JIT |
| PyPy | Python with a JIT compiler |

## Key takeaways

- A compiler produces a separate output before running. An interpreter executes the program directly or through an internal form.
- Compilers usually give faster execution and earlier error detection. Interpreters usually give faster iteration, portability, and flexibility.
- Most real systems mix both approaches.

## Common mistakes

- Thinking a language is inherently compiled or interpreted. It depends on the implementation.
- Thinking "interpreted" means no translation happens. Interpreters often transform source into a tree or bytecode first.
- Assuming a compiled program can't contain interpreted parts. A compiled program can embed an interpreter, such as Lua inside a game engine.
