# Elixir Mix and OTP: Routing Guide

Use this reference for Mix projects, OTP applications, state-owning processes,
supervision, tasks, distribution, configuration, and releases. Read the narrowest
official page that covers the task before choosing an abstraction or command.

**Source index:** [Elixir `llms.txt`](https://elixir.hexdocs.pm/llms.txt)  
**Version observed:** Elixir v1.20.3  
**Reviewed:** 2026-08-10

Check the project's Elixir and Erlang/OTP constraints before applying APIs or release
instructions. The guide version may advance after this reference is reviewed.

## Page routing

| Read | When |
|---|---|
| [Introduction to Mix](https://elixir.hexdocs.pm/introduction-to-mix.md) | Creating a project, understanding `mix.exs`, compiling, testing, formatting, Mix environments, or discovering Mix tasks. |
| [Simple state with agents](https://elixir.hexdocs.pm/agents.md) | Encapsulating simple process-owned state, choosing `Agent`, or testing stateful processes. |
| [Registries and supervision trees](https://elixir.hexdocs.pm/supervisor-and-application.md) | Distinguishing Mix projects from OTP applications, naming dynamic processes with `Registry`, or defining application startup and static supervision. |
| [Supervising dynamic children](https://elixir.hexdocs.pm/dynamic-supervisor.md) | Defining child specifications, starting runtime-created children, using `DynamicSupervisor`, or supervising processes in tests. |
| [`Task` and `gen_tcp`](https://elixir.hexdocs.pm/task-and-gen-tcp.md) | Running concurrent work under `Task.Supervisor`, designing socket ownership, or selecting child restart behavior. |
| [Doctests, patterns, and `with`](https://elixir.hexdocs.pm/docs-tests-and-with.md) | Adding doctests, parsing commands with patterns, sequencing tagged outcomes with `with`, or writing realistic integration tests. |
| [Configuration and distribution](https://elixir.hexdocs.pm/config-and-distribution.md) | Separating build/runtime configuration, managing dependencies, connecting BEAM nodes, or choosing local versus distributed process naming. |
| [Client-server with `GenServer`](https://elixir.hexdocs.pm/genservers.md) | Moving beyond simple state to calls, casts, monitors, subscriptions, or arbitrary process messages. |
| [Releases](https://elixir.hexdocs.pm/releases.md) | Building, configuring, starting, connecting to, or customizing a production release. |

## Choose the smallest runtime abstraction

| Need | Start with | Key boundary |
|---|---|---|
| Pure transformation or code organization | Module and functions | Do not introduce a process without a runtime property such as state ownership, concurrency, resource access, or failure isolation. |
| Simple process-owned state | `Agent` | Encapsulate access behind one module; callbacks execute in the agent and expensive work blocks it. |
| Client-server behavior or custom messages | `GenServer` | Use calls for synchronous requests and back pressure; use casts sparingly because callers receive no reply or receipt guarantee. |
| One-off concurrent work | `Task` | Supervise production tasks when their lifecycle matters. Closures copy captured data across process boundaries. |
| Concurrent dynamic tasks | `Task.Supervisor` | Keep failures from cascading into unrelated workers or the caller. |
| Known startup children | `Supervisor` | Define the static tree and deliberate startup order, shutdown order, and restart strategy. |
| Runtime-created children | `DynamicSupervisor` | Use it instead of growing a static supervisor to manage a large dynamic child set. |
| Local process lookup | `Registry` | Registry names are node-local and can use arbitrary Elixir terms. |
| Cross-node lookup | Distribution-specific mechanism | `:global` requires connected nodes and has scaling/partition trade-offs; do not treat it as a transparent Registry replacement. |

Before adding any of these, also check the [process-related anti-patterns](anti-patterns.md#process-related-anti-patterns).

## Mix project workflow

Inspect `mix.exs` before running or changing commands. It defines project metadata,
application behavior, and dependencies.

```bash
mix help
mix help <task>
mix compile
iex -S mix
mix format <touched-files>
mix test <relevant-test-files>
```

Useful broader gates when the repository has not defined alternatives:

```bash
mix format --check-formatted
mix test
```

Key rules:

- `mix test` uses the `:test` environment; normal development defaults to `:dev`.
- Mix is a build tool and is not expected in production. Use `Mix.env/0` only in
  configuration files or `mix.exs`, never in application code under `lib`.
- Discover tasks with `mix help`; do not guess project aliases.
- Use `mix deps.get` and `mix deps.update` deliberately, and commit `mix.lock` for
  repeatable application builds.
- After changing a supervision tree, restart `iex -S mix`; `recompile()` does not
  reload the running tree.

## Applications and supervision

A Mix **project** organizes compilation, testing, and dependencies. An OTP
**application** groups runtime modules and dependencies and may start a supervision
tree through an `Application.start/2` callback.

- Start long-running processes under a supervisor; direct `start_link` calls are
  normally reserved for the root or encapsulated child startup.
- Child order is startup order; shutdown happens in reverse order.
- Choose restart semantics intentionally: `:permanent`, `:transient`, or `:temporary`.
- `use Agent`, `use GenServer`, and `use Supervisor` provide default `child_spec/1`
  implementations, but verify IDs, start arguments, and restart behavior.
- Use `start_supervised/1` in ExUnit instead of manually starting linked children so
  the test supervisor owns cleanup.
- Use `:observer.start()` for local introspection when Erlang WX support is installed.

## Agent and GenServer boundaries

Keep the public API and process protocol in one module. Callers should not scatter raw
`Agent`, `GenServer.call`, or `GenServer.cast` operations across the system.

- `Agent.get/2`, `Agent.update/2`, and `Agent.get_and_update/2` execute their callbacks
  in the server process. Move expensive computation outside when possible.
- `GenServer.call/2` is synchronous and naturally applies back pressure.
- `GenServer.cast/2` is asynchronous: there is no reply and no receipt guarantee.
- `handle_info/2` handles messages that do not come through call/cast, including
  monitor `:DOWN` messages and ordinary `send/2` messages.
- Links couple failures; monitors observe another process without coupling exits.
- Each server handles requests sequentially. Avoid making a single process an
  accidental throughput bottleneck.

## Tasks, messages, and sockets

- Start production tasks under a supervisor when they must be tracked or isolated.
- Extract only the needed fields before spawning a closure; captured variables are
  copied to the new process.
- Separate accepting connections from serving clients.
- A socket has an owning process. Transfer ownership with
  `:gen_tcp.controlling_process/2` when a worker takes over a client socket.
- Put critical acceptors after their dependencies in the child list and choose their
  restart behavior explicitly.
- Treat the guide's simple TCP server as instructional, not production-ready; real
  servers commonly use acceptor pools and dedicated libraries.

## Tests and documentation

- ExUnit can run tests asynchronously only when they do not share global names, ports,
  files, databases, or other mutable external resources.
- Use unique process names in async tests.
- Doctests are documentation first and tests second; they supplement behavioral tests.
- Prefer integration tests through public interfaces over mocks when setup remains
  deterministic and isolated.
- A bodiless function head can provide stable argument names and documentation for a
  multi-clause function.
- Use `with` to sequence matching success values and return the first non-match; avoid
  a complex ambiguous `else` block by normalizing errors near their source.

## Configuration and distribution

- `config/config.exs` runs at build time before application and dependency code loads.
- `config/runtime.exs` runs after compilation and is the normal place for environment
  variables and external runtime settings.
- Read required runtime settings with `Application.fetch_env!/2` so missing values fail
  during boot.
- Use `Application.compile_env/2` only for values genuinely required at compile time.
- Use `iex --sname name` for local distributed development and fully qualified
  `--name` nodes in production contexts that require them.
- Connected nodes must share the Erlang cookie.
- PIDs and message passing work across connected nodes, but naming, network partitions,
  data durability, and cluster discovery still require explicit design.
- A local `Registry` does not become distributed automatically.

## Releases

Build a production release with:

```bash
MIX_ENV=prod mix release
```

A release packages application code, dependencies, and an Erlang runtime so the target
does not need source code or a separate Elixir/Erlang installation.

Common generated commands include:

```bash
bin/<app> start
bin/<app> start_iex
bin/<app> stop
bin/<app> remote
bin/<app> rpc 'Module.function()'
```

Release cautions:

- Build on the same operating-system distribution and version as the target.
- Keep runtime values in `config/runtime.exs` or supported release environment files.
- Releases use `RELEASE_NODE` and `RELEASE_DISTRIBUTION`; they do not accept `--sname`.
- Do not hard-code `-name`, `-sname`, `-setcookie`, or `-mode` in `rel/vm.args.eex`.
- Run `mix release.init` only when customization templates are actually needed.
