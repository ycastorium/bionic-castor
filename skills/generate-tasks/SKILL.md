---
name: generate-tasks
description: Use when an approved design.md exists (from the brainstorming skill) and you are about to implement it; produces the task list the implementing-tasks skill executes.
---

# Generate Tasks

Turn an approved `design.md` into an ordered list of **commit-sized tasks**. Each task is one self-contained commit that leaves the build green. A task says *what* to implement and *where*, pointing at real files and patterns rather than spelling out every line. Each implementer will see only its own task, so a task must carry everything it needs.

**Announce at start:** "I'm using the generate-tasks skill to break the design into tasks."

**Input:** the approved `design.md` in the project's Obsidian vault, usually `specs/YYYY-MM-DD-<topic>/design.md`. If you don't have the path, ask. Do not start from a verbal description; if there is no design, use the brainstorming skill first.

**Output:** `tasks.md` next to the design, built from `tasks-template.md` in this skill directory. (User preferences for location override this.)

## Process

1. **Read the whole design.** Goals bound the work, Constraints bind it hard, the Components and Flows sections define the shape.
2. **Find real code pointers.** Use the codebase graph, ast-grep, or ripgrep to locate the modules, functions, and patterns each task touches. Every `Modify:`/`Reference:` path must exist; `Create:` must be genuinely new. Never invent paths.
3. **Copy Global Constraints.** Copy the design's Constraints that bind implementation (version floors, naming rules, exact values, compatibility rules) verbatim into the `## Global Constraints` section. Every task implicitly includes them.
4. **Decompose into commit-sized tasks.** One coherent change, typically a handful of files, that compiles and passes tests on its own. Fold setup, config, scaffolding, and docs into the task whose deliverable needs them. Split only where a reviewer could reject one task while approving its neighbor. Several tiny same-shape edits across files (a rename, a constant change, a field added everywhere) are one task, not N.
5. **Write the Interfaces block.** For each task: `Consumes` (exact signatures it uses from earlier tasks) and `Produces` (exact names, parameter and return types later tasks rely on). This is how an implementer that sees only its task learns what its neighbours call things.
6. **Order by dependency.** `Depends on` records *hard* dependencies (cannot compile or pass without an earlier task's code); the order must form a DAG. A soft forward reference (an email task linking to a route created later) is a `Reference:` pointer, not a dependency.
7. **Mark execution waves.** Two tasks share a wave only when **both** hold: no dependency path between them, and their `Create:`/`Modify:` pointers touch disjoint files. When in doubt, separate waves: a wrong "parallel" costs merge conflicts, a wrong "sequential" only costs time. A single-task wave is normal.
8. **Write the file** from `tasks-template.md`, filling every field for every task.
9. **Self-review** (below), then **finish**.

## Each Task Must Have

- **What**: the change and the design section it satisfies.
- **Code pointers**: `Create:` / `Modify: path` (append `:lines` only when you know the range) / `Reference:` (a pattern to follow).
- **Interfaces**: `Consumes` and `Produces`, exact signatures. `none` is a valid value.
- **Acceptance criteria**: checkable conditions defining done, so the commit is reviewable.
- **Depends on**: task numbers or `none`.
- **Commit**: a suggested conventional-commit message.

## No Placeholders

These are failures; fix before finishing:

- "TBD", "add error handling", "write appropriate tests" without saying what.
- Code pointers to files you never verified.
- Acceptance criteria that are not checkable ("works correctly").
- A task consuming a symbol no earlier task produces.
- "Similar to Task N" instead of the actual content. The implementer reads tasks in isolation.

## Self-Review

Re-read the design and check:

1. **Coverage**: every Goal, component, and flow maps to at least one task; nothing serves a Non-Goal.
2. **Pointers are real**: each `Modify:`/`Reference:` exists; each `Create:` is new.
3. **Dependencies are a DAG**: nothing depends on a later task.
4. **Waves are safe**: every task in exactly one wave; no task shares a wave with a transitive dependency; no file in the `Create:`/`Modify:` pointers of two tasks in one wave.
5. **Interfaces line up**: every `Consumes` matches an earlier `Produces` by exact name and type.
6. **Global Constraints** are copied verbatim, not paraphrased.

Fix inline, then finish.

## Finishing

> Task list complete: `specs/2026-06-16-<feature>/tasks.md` (8 tasks in 3 waves, self-reviewed).
> To start building, load the **implementing-tasks** skill.
