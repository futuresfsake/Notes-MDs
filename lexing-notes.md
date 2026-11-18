# Lexing, Tokenization, and the Start of Compilation

## 1. What happens first when a compiler runs?

For most compilers, the first language-aware step is **lexing**, also called **scanning** or **lexical analysis**.

**Definition:** A lexer reads a stream of characters and groups them into **tokens**, the smallest meaningful units of a language: keywords, identifiers, numbers, operators, and punctuation. Whitespace and comments are usually discarded.

**Purpose:** The parser shouldn't have to think about individual characters. If the lexer turns `int`, `count`, `42`, and `+=` into single units, the grammar stays simple. Character-level details such as escape sequences, string literals, and number formats are handled in one place.

**How it works:**

```c
int total = price * 12; // comment
```

becomes:

```
KEYWORD(int)  IDENT(total)  ASSIGN  IDENT(price)  STAR  NUMBER(12)  SEMI
```

The lexer typically uses rules written as regular expressions, such as "an identifier starts with a letter or underscore, followed by letters, digits, or underscores." It applies the **maximal munch** rule: it takes the longest possible match, so `>=` is one token, not `>` followed by `=`. Lexers are usually deterministic finite automata, which run in linear time.

**Pipeline position:**

```
source text → LEXER (tokens) → PARSER (AST) → SEMANTIC ANALYSIS → code generation / interpretation
```

**Common mistakes:**
- Assuming tokens and AST nodes are the same. Tokens are a flat sequence; the parser turns them into a tree.
- Writing an overly clever lexer. Ambiguous token rules complicate parsing. C++ needs some feedback between lexer and parser for this reason.
- Forgetting the preprocessor in C and C++, which can produce tokens the source file never contained.

**Best practices:**
- Keep lexer rules simple and documented, and test edge cases: escape sequences, numeric literals, and unterminated strings.
- Attach source positions (line and column) to tokens so error messages can point to the exact location.

## 2. Is this the same as "tokenization" in AI?

The word is the same, and the idea is related: split raw input into smaller units that later code can process. The mechanisms differ.

- **Compiler tokens** come from a fixed, hand-specified set of rules. The same input always produces the same tokens, and each token has a defined meaning in the language.
- **LLM tokens** come from a learned vocabulary, usually built with an algorithm such as Byte-Pair Encoding (BPE), which merges frequently occurring character sequences. A word may be split into subword pieces, for example `tokenization` → `token` + `ization`. The pieces are chosen by frequency in training data, not by grammar, so they don't correspond to linguistic or syntactic units.

The compiler's tokenizer is a precise, rule-based component. The LLM's tokenizer is statistical. The term became common in AI with the spread of transformer models.

## 3. The C and C++ preprocessing step

In C and C++, a **preprocessor** runs before the compiler's main stages. It handles `#include`, `#define` macros, and `#if` conditionals, working on the source text to produce expanded text. For example:

```c
#define N 10
int a[N];
```

becomes `int a[10];` before the compiler's main stages see it.

Nuance: the C standard describes tokenization as one of several translation phases, and the preprocessor itself operates on tokens rather than raw characters. So in C, lexing and preprocessing are closely intertwined. Conceptually, though, the preprocessor is a separate step that runs before the language grammar is applied.

Many languages, including Java, Python, Go, and Rust, have no preprocessor, so their first step is lexing.

## 4. Does the lexer define the language?

Partly. The lexer defines the language's **vocabulary**: which character sequences count as keywords, identifiers, numbers, operators, comments, and string literals. Two languages with different lexical rules produce different token streams from the same characters.

The lexer does not decide how words combine into valid statements, or what the statements mean:

- **Lexer:** what the words are
- **Parser (grammar):** how the words form valid sentences
- **Semantic analysis:** whether those sentences make sense

## 5. Why have a lexer if you already have a compiler?

The lexer is part of the compiler, not an alternative to it. The compiler is the whole program that turns source code into something runnable, and the lexer is one stage inside it, like a station on an assembly line.

Splitting the work into stages has practical benefits:

- **Simplicity.** Handling whitespace, comments, and multi-character operators in every grammar rule would clutter the grammar. The lexer handles them once.
- **Speed.** Lexers are fast, usually deterministic and linear in input size.
- **Reuse.** Editors, syntax highlighters, and linters often need only tokens, not a full tree.
- **Error reporting.** Problems such as an unterminated string are easier to report at the character level.

Some parsers skip the separate lexer. "Scannerless" parsers, often built on Parsing Expression Grammars (PEGs), read characters directly. They're useful when lexical and grammar rules are tightly mixed, but they're less common because separation helps in most cases.

## Key takeaways

- For most compilers, lexing is the first language-aware step, turning characters into tokens.
- In C and C++, the preprocessor runs before the compiler's lexing and parsing, expanding macros and includes.
- The lexer defines the language's words; the grammar defines how they combine; semantic analysis checks meaning.
- The lexer is a component of the compiler. Separate stages make each part simpler and easier to test, change, and reuse.
- "Tokenization" in AI is a related idea, but LLM tokens are learned subword units, while compiler tokens follow fixed grammar rules.

## What a senior engineer would think about

Lexers are cheap to write with generator tools such as Flex or re2c, so they're rarely the expensive part of a compiler. Clarity matters more than raw speed here, because a confusing token definition causes problems in every later stage. When you read about AI tokenization, remember it's about statistical text chunks, not grammar. The two shouldn't be conflated.
