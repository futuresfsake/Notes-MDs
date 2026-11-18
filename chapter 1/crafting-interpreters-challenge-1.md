# Crafting Interpreters, Chapter 2 Challenge 1

## The challenge

> There are at least six domain-specific languages used in the little system I cobbled together to write and publish this book. What are they?

A domain-specific language (DSL) is a small language built for one job, such as styling pages, writing templates, or describing build steps. This challenge asks you to find the small languages in the book's own publishing toolchain.

## Answer

The author's list isn't in the book text. The answer below comes from the book's public GitHub repository, [munificent/craftinginterpreters](https://github.com/munificent/craftinginterpreters). The repository's `tool/` directory holds the build scripts, so treat this as a reading of the repo rather than the author's official list.

| # | Language or format | What it does in the book's build | Where it lives |
|---|---|---|---|
| 1 | **Markdown** | Holds the prose of each chapter | `book/` |
| 2 | **Snippet markers** (special comments) | Tells the generator where each code snippet goes in the text, and lets the build split out per-chapter code | Comments in `java/` and `c/` source |
| 3 | **Sass / SCSS** | Styles the generated site | `asset/` |
| 4 | **Jinja2 templates** | Generate the final HTML pages | `asset/` |
| 5 | **Makefile** | Orchestrates most build steps | Repository root |
| 6 | **Lox test format** | Lox test programs with expected-output comments, run against the interpreters | `test/` |

Dart is also part of the toolchain, but it is a general-purpose language used for the build scripts, so it doesn't count as a DSL.

## Notes

- Markdown and Sass are the most familiar entries. Snippet markers and the test format are custom conventions, which is why they're easy to miss.
- Lox itself is a language, but in this context it's the subject of the book rather than a tool for publishing it. The test files are the publishing-related use.
- The repository doesn't include a separate grammar file for `GenerateAst.java`, so I can't confirm whether it uses its own input format. If it does, it would be a seventh entry.

## Key takeaways

- A DSL is a small language built for one domain, not a general-purpose language.
- A real publishing pipeline often combines several DSLs: a markup language for content, a styling language, a template language, and a build tool's script format.
- Custom conventions, such as comment markers, can act as DSLs even when they aren't formal languages.

## Common mistakes

- Counting general-purpose languages, such as Dart or Java, as DSLs.
- Assuming a book's toolchain is a single program. Most real pipelines are several tools connected together.

## Sources

- [Crafting Interpreters, Introduction](https://craftinginterpreters.com/introduction.html)
- [munificent/craftinginterpreters repository](https://github.com/munificent/craftinginterpreters)
