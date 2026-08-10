# Elixir Anti-Patterns: Review Guide

Use this reference when reviewing or designing Elixir code, modules, process boundaries,
or macros. It routes each smell to the matching official guide section; consult the
source's example, refactoring, and additional remarks before recommending a change.

**Source index:** [Elixir `llms.txt`](https://elixir.hexdocs.pm/llms.txt)  
**Catalog introduction:** [What are anti-patterns?](https://elixir.hexdocs.pm/what-anti-patterns.md)  
**Version observed:** Elixir v1.20.3  
**Reviewed:** 2026-08-10

## Review discipline

An anti-pattern is an indicator of a potential problem, not proof that code is wrong.
The official guide explicitly warns that matching code may still be the best solution
for its context and that no codebase should aim to remove every anti-pattern.

For each suspected smell:

1. Name the specific anti-pattern instead of making a generic style complaint.
2. Explain the concrete cost present in this codebase.
3. Read the source's **Problem**, **Example**, **Refactoring**, and any **Additional
   remarks** before proposing a change.
4. Check whether the documented exceptions apply.
5. Recommend refactoring only when its benefit outweighs migration and API costs.

## Code-related anti-patterns

Full guide: [Code-related anti-patterns](https://elixir.hexdocs.pm/code-anti-patterns.md)

| Anti-pattern | Read when you see | Preferred direction |
|---|---|---|
| [Comments overuse](https://elixir.hexdocs.pm/code-anti-patterns.md#comments-overuse) | Comments narrating self-explanatory operations. | Improve names and structure; reserve comments for context that code cannot express and use first-class docs for public APIs. |
| [Complex `else` clauses in `with`](https://elixir.hexdocs.pm/code-anti-patterns.md#complex-else-clauses-in-with) | A large `with` whose failures are flattened into an ambiguous `else`. | Normalize errors near the operation that produces them and keep `with` focused on the success path. |
| [Complex extractions in clauses](https://elixir.hexdocs.pm/code-anti-patterns.md#complex-extractions-in-clauses) | Multi-clause signatures extracting many values used only in bodies. | Match only clause-selection data in the head; bind the whole value and extract body data inside. |
| [Dynamic atom creation](https://elixir.hexdocs.pm/code-anti-patterns.md#dynamic-atom-creation) | `String.to_atom/1` or equivalent on unbounded external values. | Explicitly map allowed values or use `String.to_existing_atom/1` only when atom existence is guaranteed correctly. |
| [Long parameter list](https://elixir.hexdocs.pm/code-anti-patterns.md#long-parameter-list) | High-arity functions with confusing related arguments. | Group cohesive values in maps, structs, tuples, or keyword options; split functions that own unrelated responsibilities. |
| [Namespace trespassing](https://elixir.hexdocs.pm/code-anti-patterns.md#namespace-trespassing) | A package defining modules under another package's namespace. | Prefix modules with the owning package namespace unless an explicit extension convention permits otherwise. |
| [Non-assertive map access](https://elixir.hexdocs.pm/code-anti-patterns.md#non-assertive-map-access) | `map[:key]` for required known atom keys, allowing missing data to become `nil`. | Use `map.key`, pattern matching, or an appropriate struct for required keys; keep Access syntax for optional or dynamic keys. |
| [Non-assertive pattern matching](https://elixir.hexdocs.pm/code-anti-patterns.md#non-assertive-pattern-matching) | Defensive parsing or catch-all clauses that silently accept unplanned shapes. | Match known valid shapes and outcomes explicitly so invalid states fail near their source. |
| [Non-assertive truthiness](https://elixir.hexdocs.pm/code-anti-patterns.md#non-assertive-truthiness) | `&&`, `||`, or `!` where operands must be booleans. | Use `and`, `or`, and `not` to assert boolean expectations, especially at Erlang boundaries. |
| [Structs with 32 fields or more](https://elixir.hexdocs.pm/code-anti-patterns.md#structs-with-32-fields-or-more) | Very wide structs crossing the BEAM map representation threshold. | Consider nesting cohesive or infrequently accessed fields while preserving API ergonomics. |

## Design-related anti-patterns

Full guide: [Design-related anti-patterns](https://elixir.hexdocs.pm/design-anti-patterns.md)

| Anti-pattern | Read when you see | Preferred direction |
|---|---|---|
| [Alternative return types](https://elixir.hexdocs.pm/design-anti-patterns.md#alternative-return-types) | Options that drastically change a function's return shape. | Give each return contract a distinct, clearly named function. |
| [Boolean obsession](https://elixir.hexdocs.pm/design-anti-patterns.md#boolean-obsession) | Several overlapping boolean flags encoding one state. | Model the state with atoms or a suitable composite value. |
| [Exceptions for control-flow](https://elixir.hexdocs.pm/design-anti-patterns.md#exceptions-for-control-flow) | Callers must rescue routine, expected outcomes. | Provide a non-raising tagged-tuple API and, when useful, a bang variant for fail-fast use. |
| [Primitive obsession](https://elixir.hexdocs.pm/design-anti-patterns.md#primitive-obsession) | Strings, numbers, or booleans repeatedly parsed as richer domain values. | Parse boundaries into tuples, maps, structs, or another explicit domain representation. |
| [Unrelated multi-clause function](https://elixir.hexdocs.pm/design-anti-patterns.md#unrelated-multi-clause-function) | Clauses with the same name implement unrelated business behavior. | Split them into specifically named functions or modules; retain clauses when behavior remains coherent. |
| [Using application configuration for libraries](https://elixir.hexdocs.pm/design-anti-patterns.md#using-application-configuration-for-libraries) | A library's behavior depends on one global application-environment value. | Prefer arguments, options, or user-owned child specs; review the guide's exceptions for swappable components and compile-time needs. |

## Process-related anti-patterns

Full guide: [Process-related anti-patterns](https://elixir.hexdocs.pm/process-anti-patterns.md)

| Anti-pattern | Read when you see | Preferred direction |
|---|---|---|
| [Code organization by process](https://elixir.hexdocs.pm/process-anti-patterns.md#code-organization-by-process) | A process exists only to organize pure functions or names. | Organize code with modules and functions; add processes only for runtime properties such as state ownership, concurrency, shared-resource access, or failure isolation. |
| [Scattered process interfaces](https://elixir.hexdocs.pm/process-anti-patterns.md#scattered-process-interfaces) | Raw `Agent`, `GenServer.call`, or `GenServer.cast` usage spread across callers. | Encapsulate process interaction and accepted message/state shapes behind one module API. |
| [Sending unnecessary data](https://elixir.hexdocs.pm/process-anti-patterns.md#sending-unnecessary-data) | Messages or spawned closures capture large structures to use a small field. | Extract and send only required data before crossing the process boundary. |
| [Unsupervised processes](https://elixir.hexdocs.pm/process-anti-patterns.md#unsupervised-processes) | Long-running processes are started outside a supervision tree. | Put lifecycle-managed processes under supervision for deterministic startup, shutdown, visibility, and recovery. |

## Meta-programming anti-patterns

Full guide: [Meta-programming anti-patterns](https://elixir.hexdocs.pm/macro-anti-patterns.md)

| Anti-pattern | Read when you see | Preferred direction |
|---|---|---|
| [Compile-time dependencies](https://elixir.hexdocs.pm/macro-anti-patterns.md#compile-time-dependencies) | Macro arguments accidentally turn runtime modules into recompilation dependencies. | Keep dependencies in their real runtime context and inspect them with `mix xref` before applying advanced expansion techniques. |
| [Large code generation](https://elixir.hexdocs.pm/macro-anti-patterns.md#large-code-generation) | Repeated macro expansion emits substantial executable code. | Generate a small call and move the bulk of the work into ordinary functions. |
| [Unnecessary macros](https://elixir.hexdocs.pm/macro-anti-patterns.md#unnecessary-macros) | A macro solves a problem an ordinary function or existing construct can solve. | Prefer the function or simpler language construct. |
| [`use` instead of `import`](https://elixir.hexdocs.pm/macro-anti-patterns.md#use-instead-of-import) | `use` injects only aliases/imports or hides broad, undocumented effects. | Prefer lexical `alias`/`import`; when `use` is necessary, document its public effects like a nutrition label. |
| [Untracked compile-time dependencies](https://elixir.hexdocs.pm/macro-anti-patterns.md#untracked-compile-time-dependencies) | Compile-time module names are assembled dynamically and evade compiler tracking. | Refer to complete module aliases or generate them in a way the compiler can track; verify with `mix xref`. |

## High-signal first pass

When time is limited, check these risks first:

1. Unbounded atom creation from external input.
2. Processes introduced without a runtime property and long-running processes outside supervision.
3. Large values copied in messages or captured by spawned functions.
4. Required data accessed non-assertively and expected outcomes hidden by catch-all clauses.
5. Exceptions used for routine caller-visible control flow.
6. Macros or `use` where functions, imports, or aliases would suffice.
