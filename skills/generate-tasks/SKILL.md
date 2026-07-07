---
name: generate-tasks
description: Use when you have a design document or spec and need to break it into an ordered, commit-sized task list before implementation - one task per commit, with code pointers, acceptance criteria, and dependencies
---

# Generate Tasks

## Overview

Turn a design document into an ordered list of **commit-sized tasks**. Each task is one self-contained commit that leaves the build green and the system coherent. A task describes *what* to implement and *where* in the code — it points the implementer at real files, modules, and patterns rather than spelling out every line.

**Announce at start:** "I'm using the generate-tasks skill to break the design into tasks."

**Output:** a single file in the obsidian vault of the project `specs/YYYY-MM-DD-<feature>/tasks.md` usually by the side of the design.md (link to it) built from `tasks-template.md` in this skill directory. (User preferences for location override this.)

## When to Use

- You have a design doc, spec, or brainstorming output and are about to implement it.
- You want the work tracked as discrete commits you can review one at a time.

**When NOT to use:**
- No design doc yet → use the brainstorming skill first.
- You need bite-sized TDD steps with full code in each step (failing test → code → run → commit) → use superpowers:writing-plans instead. This skill is coarser: commit-level and descriptive.

## Process

1. **Read the whole design doc.** If none was provided, ask for the path. Do not start from a vague verbal description.
2. **Find real code pointers.** Use ast-grep / ripgrep to locate the modules, functions, and patterns each task will touch. Pointers must reference files that exist (or be clearly marked `Create:`). Never invent paths.
3. **Decompose into commit-sized tasks.** Each task = one coherent change, typically a handful of files, that compiles/passes tests on its own. Not a single line; not an entire feature.
4. **Order by dependency.** Earlier tasks unblock later ones. Record `Depends on` for *hard* dependencies (a task can't compile or pass without an earlier task's code) so the order forms a DAG with no cycles. A *soft* forward reference — e.g. an email task linking to a route created later — is not a hard dependency: note it as a `Reference:` pointer and keep the hard-dependency graph acyclic.
5. **Mark execution waves.** Group tasks into waves in the `## Execution Waves` section. Two tasks may share a wave only when **both** hold: no dependency path between them in the DAG, and their `Create:`/`Modify:` pointers touch disjoint files (two tasks editing the same file will merge-conflict even if logically independent). When in doubt, put them in separate waves — a wrong "parallel" costs merge conflicts; a wrong "sequential" only costs time. A wave with a single task is normal.
6. **Write the file** from `tasks-template.md`, filling every field for every task.
7. **Self-review** (see checklist below).
8. **Finish** — tell the user the task list is done and where it lives (see Finishing below).

## Task Granularity

A good task is one commit a reviewer can understand in isolation:
- Schema/migration in one task; the code that uses it in the next.
- A new context/module function with its tests in one task.
- Wiring it into the UI/endpoint in another.

If a task can't be described without "and then also...", split it. If two tasks always change the same lines together, merge them.

## Each Task Must Have

- **What** — the change and the design-doc section it satisfies.
- **Code pointers** — exact paths; `Create:` (new file) / `Modify: path` (append `:lines` only when you know the range — new files and append-only edits have none) / `Reference:` (a pattern to follow).
- **Acceptance criteria** — verifiable conditions defining "done", so the commit is reviewable.
- **Depends on** — task numbers that must land first, or `none`.
- **Commit** — a suggested conventional-commit message.

## No Placeholders

Every field must carry real content. These are failures — fix them before finishing:
- "TBD", "add error handling", "write appropriate tests" without saying what.
- Code pointers to files you never verified exist.
- Acceptance criteria that aren't checkable ("works correctly").
- A task referencing a symbol or file no earlier task creates.

## Self-Review

After writing, re-read the design doc and check:
1. **Coverage** — every requirement/section maps to at least one task. List and fill gaps.
2. **Pointers are real** — each `Modify:`/`Reference:` path exists; each `Create:` is genuinely new.
3. **Dependencies are sound** — order is a DAG; nothing depends on a later task.
4. **Waves are safe** — every task appears in exactly one wave; no task shares a wave with one of its (transitive) dependencies; no file appears in the `Create:`/`Modify:` pointers of two tasks in the same wave.
5. **Naming consistency** — a symbol created in Task 3 is referenced by the same name later.

Fix issues inline before finishing.

## Finishing

Once the self-review passes, tell the user the task list is complete. State:
1. **Done** — the task list is written and self-reviewed.
2. **Where** — the path to the `tasks.md` file and the number of tasks.
3. **Next** — suggest loading the `implementing-tasks` skill to execute the list, which sets up an isolated workspace and drives each task through implementation, review, and progress tracking.

Example:

> Task list complete: `specs/2026-06-16-<feature>/tasks.md` (8 tasks in 3 waves, self-reviewed).
> To start building, load the **implementing-tasks** skill — it sets up an isolated workspace and drives each task through implementation, code review, and progress tracking.
