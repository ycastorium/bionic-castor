# <Feature Name> — Implementation Tasks

> Design: `<path/to/design.md>`
> Each task below is **one commit**. Implement top to bottom; respect `Depends on`.
> Tasks sharing a wave in `## Execution Waves` may be implemented in parallel.

**Goal:** <one sentence describing what this builds>

---

## Global Constraints

> Copied verbatim from the design's Constraints. Every task implicitly includes these.

- <exact rule, e.g. "Elixir 1.17+, no new hex deps">
- <exact rule>

---

## Progress

- [ ] Task 1: <title>
- [ ] Task 2: <title>
- [ ] Task 3: <title>

---

## Execution Waves

> Tasks in the same wave have no dependencies on each other and touch disjoint files. Waves run in order; a wave starts only after the previous one is fully merged and green.

- Wave 1: Task 1, Task 2
- Wave 2: Task 3

---

## Rulings

> Decisions the orchestrator made during implementation where the design or a task was ambiguous. Format: `Task N: <what was decided> — <why> — <cost if wrong>`. Empty until implementation starts.

---

## Task 1: <short imperative title>

**What:** <1-3 sentences: what exists after this commit and why. Name the design section it implements.>

**Code pointers:**
- Create: `exact/path/to/new_file.ext` — <responsibility>
- Modify: `exact/path/to/existing.ext:120-145` — <what changes here>
- Reference: `exact/path/to/example.ext` — <existing pattern to follow>

**Interfaces:**
- Consumes: none
- Produces: `MyModule.create(attrs :: map()) :: {:ok, Thing.t()} | {:error, Ecto.Changeset.t()}`

**Acceptance criteria:**
- [ ] <checkable condition: a named test passes, an endpoint returns X, a field validates>
- [ ] <checkable condition>

**Depends on:** none

**Commit:** `feat(scope): concise message`

---

## Task 2: <short imperative title>

**What:** <...>

**Code pointers:**
- Modify: `...`

**Interfaces:**
- Consumes: `MyModule.create/1` from Task 1
- Produces: none

**Acceptance criteria:**
- [ ] <...>

**Depends on:** Task 1

**Commit:** `feat(scope): concise message`

---
