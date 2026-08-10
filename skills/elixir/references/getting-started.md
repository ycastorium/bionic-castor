# Elixir Getting Started: Routing Guide

Use this file to choose the narrowest official Elixir guide page for the task. It is a
routing reference, not a substitute for the source documentation.

**Source index:** [Elixir `llms.txt`](https://elixir.hexdocs.pm/llms.txt)  
**Version observed:** Elixir v1.20.3  
**Reviewed:** 2026-08-10

Check the version constraint in the target project's `mix.exs` before applying syntax
or APIs introduced after that version. The source index tracks current documentation,
so its displayed version may change after this reference was reviewed.

## Setup and execution

| Read | When |
|---|---|
| [Introduction](https://elixir.hexdocs.pm/introduction.md) | Installing Elixir, checking Elixir/Erlang versions, using IEx, or running `.exs` scripts. |

## Values and data structures

| Read | When |
|---|---|
| [Basic types](https://elixir.hexdocs.pm/basic-types.md) | Working with numbers, booleans, atoms, `nil`, strings, operators, or type predicates. |
| [Lists and tuples](https://elixir.hexdocs.pm/lists-and-tuples.md) | Choosing between linked lists and fixed-size tuples, or handling heads, tails, and charlist-looking output. |
| [Binaries, strings, and charlists](https://elixir.hexdocs.pm/binaries-strings-and-charlists.md) | Handling UTF-8, code points, graphemes, bytes, bitstrings, binaries, strings, or Erlang charlists. |
| [Keyword lists and maps](https://elixir.hexdocs.pm/keywords-and-maps.md) | Designing options, choosing keyword lists versus maps, or accessing and updating associative data. |
| [Structs](https://elixir.hexdocs.pm/structs.md) | Defining domain data with checked fields and defaults, updating structs, or matching on struct types. |

## Matching, branching, and functions

| Read | When |
|---|---|
| [Pattern matching](https://elixir.hexdocs.pm/pattern-matching.md) | Destructuring values, understanding the match operator, pinning variables, or matching list heads and tails. |
| [`case`, `cond`, and `if`](https://elixir.hexdocs.pm/case-cond-and-if.md) | Selecting control flow, writing guards, or understanding Elixir truthiness and clause failures. |
| [Anonymous functions](https://elixir.hexdocs.pm/anonymous-functions.md) | Passing functions, captures, closures, multi-clause anonymous functions, or function name/arity notation. |
| [Modules and functions](https://elixir.hexdocs.pm/modules-and-functions.md) | Defining modules, public/private functions, guards, multiple clauses, default arguments, or `.ex` versus `.exs`. |

## Compile-time composition and extension

| Read | When |
|---|---|
| [`alias`, `require`, `import`, and `use`](https://elixir.hexdocs.pm/alias-require-and-import.md) | Resolving module names, invoking macros, importing functions, or understanding code injected by `use`. |
| [Module attributes](https://elixir.hexdocs.pm/module-attributes.md) | Adding docs, specs, behaviours, compile-time constants, or temporary compile-time storage. |
| [Protocols](https://elixir.hexdocs.pm/protocols.md) | Implementing type-based polymorphism or deciding between protocol dispatch and function clauses. |
| [Sigils](https://elixir.hexdocs.pm/sigils.md) | Using regex, string, charlist, word-list, date/time, or custom textual literals. |
| [Optional syntax sheet](https://elixir.hexdocs.pm/optional-syntax.md) | Reading or writing omitted parentheses, trailing keyword arguments, and `do`/`end` block syntax. |

## Collection processing

| Read | When |
|---|---|
| [Recursion](https://elixir.hexdocs.pm/recursion.md) | Writing recursive functions, identifying termination clauses, or processing recursive shapes directly. |
| [Enumerables and Streams](https://elixir.hexdocs.pm/enumerable-and-streams.md) | Transforming collections, composing pipelines, or choosing eager `Enum` versus lazy `Stream`. |
| [Comprehensions](https://elixir.hexdocs.pm/comprehensions.md) | Combining generators, filters, pattern matching, and collectables with `for`. |

## Runtime boundaries

| Read | When |
|---|---|
| [`try`, `catch`, and `rescue`](https://elixir.hexdocs.pm/try-catch-and-rescue.md) | Distinguishing errors, throws, and exits; defining exceptions; or deciding whether rescue is appropriate. |
| [Processes](https://elixir.hexdocs.pm/processes.md) | Understanding process isolation, spawning, message passing, links, tasks, or process state. |
| [IO and the file system](https://elixir.hexdocs.pm/io-and-the-file-system.md) | Reading or writing IO devices and files, manipulating paths, or choosing normal versus bang APIs. |
| [Erlang libraries](https://elixir.hexdocs.pm/erlang-libraries.md) | Calling Erlang modules, handling raw binaries, using Erlang formatting or crypto, or adding OTP applications. |
| [Debugging](https://elixir.hexdocs.pm/debugging.md) | Inspecting pipeline values, using `dbg`, breakpoints, tracing, or observer-style tools. |

## Documentation

| Read | When |
|---|---|
| [Writing documentation](https://elixir.hexdocs.pm/writing-documentation.md) | Writing `@moduledoc`, `@doc`, `@typedoc`, examples, doctests, metadata, or documentation-friendly function heads. |

## Baseline principles from the guide

1. Elixir data structures are immutable; operations return transformed values.
2. `=` is the match operator, and patterns are central to destructuring and control flow.
3. Functions are identified by name and arity; modules organize public and private functions.
4. Keyword lists are primarily option lists; maps are the general key-value structure.
5. Strings are UTF-8 binaries, while charlists are lists commonly encountered at Erlang boundaries.
6. Prefer `Enum` and `Stream` for routine collection processing; use recursion when its shape matters.
7. Expected outcomes commonly use tagged tuples; `try`/`rescue` is uncommon for ordinary control flow.
8. Processes are isolated, lightweight, and communicate through message passing.
9. Documentation is first-class and can include executable doctests.
10. Call Erlang libraries directly when they provide the required functionality instead of wrapping them by default.
