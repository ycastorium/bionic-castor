---
name: elixir
description: >
  Use whenever writing, editing, reviewing, debugging, or explaining Elixir code;
  working with .ex or .exs files, mix.exs, Mix, ExUnit, IEx, Erlang interoperability,
  BEAM, OTP, Phoenix, Ecto, or Elixir dependencies. Also trigger on Elixir compiler,
  formatter, test, and runtime errors.
---

# Elixir Development

Use this skill for Elixir language and project work. Keep the main workflow small;
load the narrowest reference that covers the task before relying on memory.

## Core Principles

**Inspect the project first.** Read its instructions, `mix.exs`, `.formatter.exs`,
relevant configuration, and nearby code before editing. Respect the project's Elixir
and Erlang/OTP versions, conventions, and existing abstractions.

**Use current documentation.** Treat remembered language, library, and framework APIs
as potentially outdated. Check the project version and consult its matching official
documentation before introducing or changing an API.

**Match and transform data.** Elixir data is immutable. Prefer pattern matching,
function clauses, and guards when they make valid states and control flow explicit.
Build transformed values rather than simulating mutation.

**Choose data structures deliberately.** Lists are linked lists; tuples are fixed-size
containers; maps are the general key-value structure; keyword lists are ordered option
lists with atom keys and may contain duplicate keys. Strings are UTF-8 binaries, not
charlists.

**Use collection abstractions before hand-written recursion.** Reach for `Enum` for
eager collection work and `Stream` when lazy composition is useful. Use explicit
recursion when the recursive shape is itself part of the problem.

**Model expected failures as data.** Prefer return values such as `{:ok, value}` and
`{:error, reason}` for outcomes callers are expected to handle. Reserve exceptions for
exceptional failures; do not wrap ordinary control flow in broad `try`/`rescue` blocks.

**Treat processes as isolation boundaries.** Elixir processes communicate through
messages and do not share mutable state. Before adding process machinery, determine
whether the work actually needs concurrency, state ownership, or fault isolation.

**Document public behavior.** Elixir treats documentation as first-class. Preserve
useful `@moduledoc`, `@doc`, `@typedoc`, and `@spec` contracts, and keep examples or
doctests aligned with behavior.

**Use Erlang directly when appropriate.** Elixir interoperates with Erlang modules;
do not add a wrapper that merely renames an Erlang API without adding a meaningful
project boundary.

## Workflow

1. Read project-level instructions and identify the relevant app, module, tests, and
   configured Elixir/Erlang versions.
2. Read the narrowest relevant reference:
   - [getting-started](references/getting-started.md) for language fundamentals;
   - [mix-and-otp](references/mix-and-otp.md) for projects, applications, processes,
     supervision, distribution, and releases;
   - [anti-patterns](references/anti-patterns.md) when reviewing design, process
     boundaries, or metaprogramming.
3. Check current official documentation for any Mix, OTP, framework, or dependency API
   not covered by that reference.
4. Follow nearby code style and make the smallest coherent change.
5. Format touched files and run the narrowest relevant tests first.
6. Run broader project gates only when required by the repository or the scope of the
   change. Run Credo, Dialyzer, or framework-specific checks only when configured.

When the repository does not define commands, common defaults are:

```bash
mix format <touched-files>
mix test <relevant-test-files>
```

Use `mix test` as the broader test gate. Do not guess task aliases or add dependencies
without inspecting `mix.exs` and the project's documentation.

## Reference Routing

- **Language fundamentals:** [getting-started](references/getting-started.md) — route
  syntax, data structures, modules, collections, errors, processes, IO, documentation,
  Erlang interoperability, and debugging questions to the relevant official guide page.
- **Mix and OTP:** [mix-and-otp](references/mix-and-otp.md) — route project tooling,
  application startup, process abstractions, supervision, tests, configuration,
  distribution, and release work to the relevant official guide page.
- **Code and architecture review:** [anti-patterns](references/anti-patterns.md) — identify
  and assess official code, design, process, and metaprogramming anti-patterns without
  treating every smell as an automatic rewrite requirement.

Add focused references incrementally instead of
turning this file into a comprehensive handbook.
