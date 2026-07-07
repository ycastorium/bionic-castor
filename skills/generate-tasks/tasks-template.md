# <Feature Name> — Implementation Tasks

> Generated from design doc: `<path/to/design.md>`
> Each task below is **one commit**. Implement top to bottom; respect `Depends on`.
> Tasks sharing a wave in `## Execution Waves` may be implemented in parallel.

**Goal:** <one sentence describing what this builds>

---

## Progress

- [ ] Task 1: <title>
- [ ] Task 2: <title>
- [ ] Task 3: <title>

---

## Execution Waves

> Tasks in the same wave have no dependencies on each other and touch disjoint files — they can be implemented in parallel. Waves run in order; a wave starts only after the previous one is fully merged and green.

- Wave 1: Task 1, Task 2
- Wave 2: Task 3

---

## Task 1: <short imperative title>

**What:** <1-3 sentences: what needs to exist after this commit, and why. Reference the design doc section it implements.>

**Code pointers:**
- Create: `exact/path/to/new_file.ext` — <responsibility>
- Modify: `exact/path/to/existing.ext:120-145` — <what changes here>
- Reference: `exact/path/to/example.ext` — <existing pattern to follow>

**Acceptance criteria:**
- [ ] <verifiable condition — a test passes, an endpoint returns X, a field validates>
- [ ] <verifiable condition>

**Depends on:** none

**Commit:** `feat(scope): concise message`

---

## Task 2: <short imperative title>

**What:** <...>

**Code pointers:**
- Modify: `...`

**Acceptance criteria:**
- [ ] <...>

**Depends on:** Task 1

**Commit:** `feat(scope): concise message`

---
