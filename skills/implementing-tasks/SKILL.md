---
name: implementing-tasks
description: Use when a tasks.md task list (from the generate-tasks skill) exists and you are ready to write the code.
---

# Implementing Tasks

Work a `tasks.md` to completion, one commit-sized task at a time. Every task runs the same loop: **implement → review → fix loop → commit → mark done**. You (this session) orchestrate; the flow files decide who writes the code.

**Announce at start:** "I'm using the implementing-tasks skill to implement the task list."

**Input:** `tasks.md` and its sibling `design.md` in the project's Obsidian vault, usually `specs/YYYY-MM-DD-<feature>/`. Ask for the path if you don't have it. No task list → use generate-tasks first. A single ad-hoc change with no list → just do it.

**Narration:** one short line between tool calls at most. The task list and tool results carry the record. Do not pause between tasks to ask "should I continue?"; the user asked you to implement the list.

## Detect the Version Control System

Check for `.jj/` at the repo root (`jj root` succeeds). If present, announce it, load the **jujutsu** skill, and use jj for every write even in a colocated repo. The loop is the same; only commands change:

| Step | git | jj |
|------|-----|----|
| Feature line | `git switch -c feat/<x>` | build a stack, `jj bookmark create feat/<x> -r @`, re-`set` as it grows |
| Isolated workspace | `wt switch --create feat/<x>` under `.workspaces/` | `jj workspace add .workspaces/<x>` |
| Record BASE before dispatch | `git rev-parse HEAD` | `jj log -r @- --no-graph -T change_id` (make `@` empty first with `jj new`) |
| Commit a task | `git add -A && git commit -m "..."` | `jj commit -m "..."` |
| Merge a parallel task into base | `wt merge` | `jj rebase -s <task> -d <base>`, then `jj bookmark set` |
| Open a PR at the end | `git push` + `gh pr create` | jujutsu skill, `references/git-interop.md` |

## The Two Decisions

Ask both via `AskUserQuestion` before any code. They are orthogonal: branching is *where* work lands, execution is *how* tasks get done.

**Branching mode:**

- **Feature Branch**: keep the current non-default branch, or create `feat|bug|chore/<feature>`. On jj: a stack with a bookmark on the tip.
- **Worktree**: `wt switch --create feat/<feature>` (worktrunk places it per project config and runs post-create hooks) or `jj workspace add`. Place workspaces under `.workspaces/` in the repo root (git-ignored; the `workspace` script writes the ignore file). Warm the new workspace's build caches from the base so the first build is incremental, not cold: APFS clone `cp -c -R` on macOS, `cp --reflink=auto` on Linux, symlink as fallback. Elixir `_build/ deps/`, Rust `target/`, Node `node_modules/`, Python `.venv/`. Clone for mutable build dirs, symlink is fine for lockfile-restorable dirs. Skip whatever worktrunk hooks already populated.

Either way you get one **base workspace** where every task ultimately lands. Never implement on `main`/`master` without explicit consent.

**Execution mode:**

- **Pair Programming** → `pair-programming-flow.md`. You write the code; review is delegated.
- **Sequential with Subagents** → `sequential-subagents-flow.md`. One implementer at a time in the base, list order, waves ignored.
- **Parallel with Subagents** → `parallel-subagents-flow.md`. Tasks in a wave run concurrently in throwaway worktrees and merge back per wave.

If the user picks Parallel and `tasks.md` has no `## Execution Waves`, derive waves (no dependency path **and** disjoint `Create:`/`Modify:` files) and confirm them with the user.

## Setup

1. **Artifacts directory**: run `scripts/workspace <tasks.md>` from this skill directory. It prints a git-ignored `.workspaces/artifacts/<feature>/` in the base repo for briefs, reports, and review packages. Run every script from the base workspace.
2. **Resume check**: `tasks.md` is the durable ledger. Checked boxes in `## Progress` are done; resume at the first unchecked task. After a context compaction, trust `tasks.md` and `git log`/`jj log` over your memory. Re-dispatching a completed task is the most expensive failure there is.
3. **Read `design.md` once** and `tasks.md`'s `## Global Constraints`. You will not paste them into dispatches; the brief carries them.
4. **Pre-flight scan**: for every pair of tasks sharing a file or interface, check that what one `Produces` matches what the other `Consumes`; for every task, that its text agrees with itself. Rule on any conflict (see Rulings) before Task 1. If clean, say nothing.
5. **Seed the native task list**: one tracked task per `## Progress` entry, same order and titles. Keep it in sync; if the tool is unavailable, skip silently.

## Rulings, Not Stalls

Ambiguities, task defects, and conflicts between the design and the code are yours to decide. `design.md` is the authority, `tasks.md` is its argument, your judgment settles what neither answers. Record every decision in `tasks.md` under `## Rulings` as `Task N: <what> — <why> — <cost if wrong>` and keep going. A wrong ruling costs rework the user can see and undo; a session parked on a question costs their day.

Only these stop you and go to the user via `AskUserQuestion`: a destructive or irreversible operation; a security-sensitive choice; a side effect outside the workspace (merge, push, publish); a defect so deep that every path forward is a guess. In Pair Programming mode the user is present, so ask more freely, but still record the answer as a ruling.

When a subagent pings you with a question, answer it from the docs or rule on it. Never let an agent guess, and never leave it waiting.

## Model Selection

Always specify the model explicitly when dispatching; an omitted model inherits the session's, usually the most expensive. Turn count beats token price: the cheapest models take 2-3x the turns on multi-step work. Use a mid-tier model as the floor for implementers working from prose and for reviewers.

| Work | Model |
|------|-------|
| 1-2 files, brief spells out exactly what to write | cheap |
| Multi-file, integration concerns, or judgment | standard |
| Design judgment, broad codebase understanding, final whole-branch review | most capable |
| Scoped re-review of a small fix diff | cheap to mid |
| Fix loop rounds 4-5 | one tier above the stuck implementer |

Prefer a specialised agent type (language or domain) over a generic one when the harness offers it.

## The Per-Task Loop

For each unchecked task, in dependency order:

1. **Mark in progress** in the native list. Record **BASE** (see the VCS table).
2. **Brief**: `scripts/task-brief <tasks.md> N` writes `task-N-brief.md` (Global Constraints + the task) and prints the path. The brief is the implementer's only requirements source. Never make an implementer read the whole `tasks.md` or paste task text into a prompt.
3. **Implement** per the chosen flow. Subagents get: one line on where the task fits, the brief path, interfaces and rulings from earlier tasks the brief cannot know, and the report path `task-N-report.md`. Template: `implementer-prompt.md`. Implementers follow the **tdd** and **ponytail** skills, never spawn subagents, and return under 15 lines; the report file holds the detail.
4. **Handle the report**: `DONE` → review. `DONE_WITH_CONCERNS` → read the concerns; correctness or scope concerns get addressed before review. `NEEDS_CONTEXT` → provide it and resume. `BLOCKED` → change something: more context, a stronger model, a smaller task, or a ruling on a task defect. Never re-run the same model with the same input.
5. **Review**: `scripts/review-package <tasks.md> BASE HEAD` writes the diff file. Dispatch `task-reviewer-prompt.md` with the brief, the report, and the diff path. The reviewer checks acceptance criteria, correctness, then complexity in the ponytail-review format. It does not re-run the suite; the report carries the RED/GREEN evidence. Do not pre-judge findings for it ("don't flag X" is you dodging a review round). `⚠️ Cannot verify` items are yours to resolve with your cross-task context.
6. **Fix loop**: triggers on ❌ acceptance, any Critical or Important finding, or a ⚠️ you confirmed. Minors go to `## Rulings` as `Task N: minor (deferred): <one-liner>` and never enter the loop. A finding that conflicts with the task text gets a ruling first. One round = one fix dispatch + one scoped re-review (`review-package` over FIX_BASE..HEAD, `re-review-prompt.md`), five rounds max. Rounds 1-3 resume the original implementer with the findings verbatim; rounds 4-5 dispatch a fresh implementer on a stronger model with the brief, the report, and "a prior implementer attempted this N times; read the report for what was tried". At round 5 with findings still open, stop dispatching and adjudicate each one: park it with a ruling (reviewer wrong, or real but nothing builds on it) or rule on the smallest change that unblocks dependent work and carry it into the next dispatch. Every adjudication is a ruling line; silent discards are forbidden. In Pair Programming mode you are the implementer, so you fix and the scoped re-review still runs.
7. **Commit** with the task's suggested message: the implementer commits in subagent modes and you verify the commit exists; you commit in Pair Programming. Review fixes are amended in so the task stays one commit (jj: `jj squash` into the change).
8. **Mark done**: native list `completed`, and in `tasks.md` check the `## Progress` box and the acceptance criteria. A task is done only when reviewed, committed, and checked off in both places.

Parallel mode runs steps 1-7 concurrently per wave, each task in its own worktree; step 8 happens after the task is merged into the base and the base is green.

## Final Review

After the last task: `scripts/review-package <tasks.md> MERGE_BASE HEAD` (the commit the branch started from) and dispatch `task-reviewer-prompt.md` in whole-branch mode on the most capable model, with `design.md` and `tasks.md` as the requirements and a pointer to the deferred minors and parked rulings so it can triage what must be fixed before merge. If it returns findings: ONE fix dispatch with the complete list, one scoped re-review, then adjudicate residuals as in the loop. There is no second fix wave.

## Once All Tasks Are Done

Confirm every `## Progress` box is checked, every native task is completed, and the full suite is green in the base (run it now; a previous run is not evidence). Then report to the user:

1. Done, and where the branch is.
2. **Rulings I made**: every `## Rulings` line, in order, each with its cost if wrong. This is the only place decisions taken on the user's behalf reach them.
3. Ask for next steps (PR, merge, finish the branch). Do not push or merge unless asked. On jj, set the bookmark on the tip first.

Delete `.workspaces/artifacts/<feature>/` once the final review is clean; git history is the record now.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "I'll pick a mode myself, asking is overhead" | Always ask both decisions. They change where code lands and who writes it. |
| "I'll fix the finding myself, dispatching is overhead" | Controller fixes skip review and pollute your context. Resume the implementer. |
| "The fix was tiny, skip the re-review" | Unreviewed fixes are how regressions land. Every round ends with a scoped re-review. |
| "One more round will converge" | Past five, rounds don't converge; the failure is structural. Adjudicate and ledger. |
| "This finding is obviously wrong, drop it" | Adjudicate only at the cap, and every ruling is a line in `## Rulings`. |
| "I'll ask the user, they should decide" | Only the four stop conditions go to the user. Everything else is a ruling you record. |
| "I remember where we were" | After compaction you don't. Read `tasks.md` and the log. |
| "Implementer said done, mark it" | Reviewed, committed, checked off, in that order. Agent reports are claims until the reviewer and the diff confirm them. |
| "The implementer spawned a reviewer, free assurance" | Duplicate seat, same diff. The task review is the gate; a worker-spawned reviewer is a defect to flag. |
| "These tasks look independent, run them together" | Only `## Execution Waves` (or a user-confirmed derivation) defines concurrency. |
| "Wave N is mostly merged, start N+1" | Every wave-N task merged and the base green first. Never start a wave on a broken base. |
