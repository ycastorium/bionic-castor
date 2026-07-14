---
name: jujutsu
description: Use when working in a repository managed by Jujutsu (the `jj` CLI) instead of, or alongside, plain git. Triggers on "use jj", "jujutsu", "jj log/status/commit/describe/squash/split/rebase", "colocated git repo", "jj workspace", "jj bookmark", "jj git push", "create a PR with jj", "the working copy is a commit", "revset", "undo in jj", or when a repo has a `.jj/` directory. Covers the mental model, the core agent workflow, editing history, bookmarks-as-branches, colocated git repos, workspaces, and recovery.
---

# Working with Jujutsu (`jj`)

Jujutsu is a git-compatible VCS with a different, simpler model. This skill teaches you to drive `jj` as an agent: how to make changes, shape history, sync with git remotes, use workspaces, and recover from mistakes — without falling back on git muscle memory that fights the model.

If a repo has both `.jj/` and `.git/`, it is a **colocated** repo: jj and git share the same commits, and you may use either tool. Prefer `jj` for anything that writes history; use `git` only for read-only inspection or the rare thing jj can't do. See `references/git-interop.md`.

## The mental model (read this first)

Four ideas explain almost every difference from git. Internalize them before running commands.

1. **The working copy is a commit.** There is no staging area and no "dirty tree". Your uncommitted edits live in a real commit called `@` (the working-copy commit). Every time you run a `jj` command, jj snapshots your files into `@` automatically. You never `git add`. "Committing" in jj mostly means *giving `@` a description and starting a new empty `@` on top*.

2. **Changes have a stable change ID.** Every commit has two identifiers: a **change ID** (e.g. `qpvuntsm`, shown in the left color, letters only) that stays constant as you rewrite the commit, and a **git commit hash** (hex) that changes on every rewrite. Refer to commits by their **change ID** — it survives amends, rebases, and squashes. jj highlights the shortest unique prefix; you can type just that prefix.

3. **Rewriting is normal and safe.** Amending, splitting, squashing, and rebasing are everyday operations. When you rewrite a commit, jj **automatically rebases all its descendants** onto the new version — no manual `rebase --onto` chains. Conflicts don't block this: they get recorded in the commits and you resolve them when convenient.

4. **Every operation is undoable.** jj records every repo-modifying command in an **operation log**. `jj undo` reverses the last one; `jj op restore <id>` jumps the whole repo back to any prior state. You can experiment freely.

## Golden rules for agents

- **Orient before acting.** Run `jj status` and `jj log` before and after changes. Never assume where `@` is.
- **Refer to commits by change ID**, not git hash — hashes move under you.
- **You do not stage or `jj add`.** Just edit files; jj snapshots them into `@`.
- **Give work a description early.** Run `jj describe -m "..."` on `@` so the change isn't an anonymous "(no description)".
- **Move the bookmark before you push.** Bookmarks (jj's branches) do NOT auto-follow new commits. After committing, point the bookmark at the new commit, then push. This is the single most common agent mistake.
- **Don't rewrite immutable commits.** Ancestors of `trunk()` (usually `main`/`master`) and tagged commits are immutable by default; jj refuses to rewrite them. That's a guardrail, not a bug — build on top instead.
- **When something looks wrong, `jj undo` — don't improvise.** The op log makes recovery trivial; see `references/operations-and-recovery.md`.
- **In a colocated repo, prefer `jj` for writes.** Mixing `git commit`/`git rebase` with jj works but creates avoidable divergence and imported "operations". Read with git if you like; write with jj.

## Orientation commands

```
jj status              # what changed in @, and @'s parent(s)
jj log                 # graph of relevant commits (defaults to @ + ancestors to trunk + other heads)
jj log -r 'all()'      # everything (can be large)
jj diff                # diff of @ against its parent (your current uncommitted work)
jj diff -r <change>    # diff of a specific commit
jj show <change>       # message + diff for a commit
jj op log              # history of operations you've run (for undo/recovery)
```

Reading `jj log`: `@` marks the working-copy commit. Each node shows change ID, author, timestamp, bookmarks, and description. `(empty)` means no file changes; `(no description)` means undescribed.

## Core workflow: making changes

The default loop. `@` always holds your in-progress edits.

```
# 1. Start a fresh change on top of the current commit (or a specific one)
jj new                      # new empty @ on top of current @
jj new <change>             # new empty @ on top of <change>
jj new main                 # start work on top of the main bookmark

# 2. Edit files normally. jj snapshots them into @ automatically — no add/stage.

# 3. Describe the work (do this early; you can re-run to refine)
jj describe -m "Add rate limiter to API gateway"

# 4a. Finish and start the next change in one step:
jj commit -m "Add rate limiter to API gateway"
#   -> sets @'s message and creates a new empty @ on top. Equivalent to
#      `jj describe -m ...` followed by `jj new`.

# 4b. Or just keep editing @ — there is no separate "commit" needed to save work;
#     it's already a real commit. Use `jj commit`/`jj new` only to START A NEW change.
```

**Stacking changes** (a chain of dependent commits, the natural jj way to build a feature):

```
jj commit -m "Step 1: add config schema"
jj commit -m "Step 2: wire config into loader"
jj commit -m "Step 3: use config in gateway"
# Each `jj commit` finalizes the current change and opens the next on top.
# `jj log` now shows a clean stack; each is independently reviewable and rebaseable.
```

## Editing history

All of these rewrite commits and auto-rebase descendants. Refer to targets by change ID.

```
# Amend @ into its parent (like `git commit --amend`): move @'s changes down
jj squash                      # squash all of @ into @-
jj squash -i                   # interactively pick which hunks to move
jj squash --into <change>      # move @'s changes into an arbitrary commit
jj squash --from <A> --into <B># move changes from A into B

# Amend a NON-working commit: edit its files directly
jj edit <change>               # point @ at <change>; your edits now rewrite it
#   (When done, `jj new <child-or-top>` to stop editing it and move on.)

# Change a message without touching files
jj describe <change> -m "Better message"

# Split one commit into two
jj split                       # interactive: choose what stays vs. goes to a new commit
jj split <paths>...            # non-interactive: listed paths become the first commit

# Drop a commit, rebasing its descendants over the gap
jj abandon <change>

# Reorder / re-parent commits
jj rebase -r <change> -d <dest>    # move a single commit onto dest
jj rebase -s <change> -d <dest>    # move <change> AND its descendants onto dest
jj rebase -b <change> -d <dest>    # move the whole branch containing <change>

# Pull specific files back from another commit into @
jj restore --from <change> <paths>...
```

If jj refuses with an "immutable" error, you're trying to rewrite a protected commit (ancestor of `trunk()`, or tagged). Build on top instead; only if you truly must, pass `--ignore-immutable` — and don't do that to shared/pushed history.

## Conflicts

jj does not stop the world on a conflict. A rebase/squash that conflicts still succeeds; the conflict is **stored inside the resulting commit** and shown in `jj log`/`jj status`. Work continues; resolve when ready.

```
jj status                      # shows which files/commits have conflicts
jj resolve                     # launch the configured merge tool for conflicted files
jj resolve --list              # list conflicts without opening a tool
# Or open the files: jj writes standard conflict markers you can edit by hand,
# then jj auto-detects resolution on the next snapshot.
```

Because conflicts live in commits, you can resolve a conflict in the commit where it *originated* (via `jj edit <change>` then `jj resolve`) and let jj propagate the fix to descendants.

## Bookmarks (jj's branches)

jj calls named pointers **bookmarks**. They map to git branches on push/fetch. The critical difference from git: **a bookmark does not move when you create new commits.** You move it explicitly.

```
jj bookmark list                          # show bookmarks (local and remote-tracking)
jj bookmark create feature-x -r @         # create a bookmark at @
jj bookmark set feature-x -r @            # move an existing bookmark to @
jj bookmark move feature-x --to @         # same idea, alternate syntax
jj bookmark delete feature-x              # delete a local bookmark
jj bookmark rename old new
jj bookmark track main@origin             # track a remote bookmark locally
```

Moving a bookmark **backward** (to an ancestor) needs `--allow-backwards`. After you commit new work on a branch, remember: `jj bookmark set <name> -r @` before pushing.

## Colocated git repos (summary)

A colocated repo has `.jj/` and `.git/` side by side. jj auto-imports git refs and auto-exports jj bookmarks around each command, so `git log`/`git status` stay meaningful and you can hand the repo to git-only tools. Create one with `jj git init --colocate` (in an existing git repo) or `jj git clone --colocate <url>`.

Full details — pushing, fetching, PRs, bookmark↔branch mapping, and when to reach for `jj git import`/`export` — are in **`references/git-interop.md`**. Read it before pushing or opening a PR.

## Workspaces (summary)

Workspaces are multiple working copies backed by one repository — jj's answer to `git worktree`, but each workspace has its own independent `@`. Use them to work on two changes at once without stashing.

```
jj workspace add ../feature-y            # new working copy in a sibling dir
jj workspace list
jj workspace update-stale                # fix a "stale working copy" after edits elsewhere
```

Details, naming, cleanup, and the stale-working-copy gotcha are in **`references/workspaces.md`**.

## Recovery (summary)

```
jj undo                     # reverse the last operation
jj op log                   # see every operation, with IDs
jj op restore <op-id>       # reset the whole repo to that operation's state
```

Nothing you do is truly lost until garbage collection; abandoned commits and old states are reachable through the op log. Full recovery playbook in **`references/operations-and-recovery.md`**.

## git → jj translation table

| Goal | git | jj |
|------|-----|----|
| See status | `git status` | `jj status` |
| See history | `git log --graph` | `jj log` |
| Stage + commit | `git add -A && git commit -m` | `jj commit -m` (no staging) |
| Amend last commit | `git commit --amend` | `jj squash` (from `@`) or `jj describe` |
| New branch | `git switch -c x` | `jj bookmark create x -r @` |
| Switch/checkout | `git switch x` | `jj new x` or `jj edit <change>` |
| Rebase | `git rebase main` | `jj rebase -d main` (descendants auto-follow) |
| Cherry-pick | `git cherry-pick <c>` | `jj rebase -r <change> -d @` or `jj duplicate <change>` |
| Undo a rebase/commit | `git reflog` + reset | `jj undo` / `jj op restore` |
| Discard file changes | `git restore <f>` | `jj restore <f>` |
| Push branch | `git push -u origin x` | `jj git push --bookmark x` |
| Fetch | `git fetch` | `jj git fetch` |

## References

Load these on demand — don't inline them unless the task needs the depth:

- **`references/git-interop.md`** — colocated repos, `jj git push`/`fetch`/`import`/`export`, bookmarks↔branches, pushing changes, and the end-to-end "make a PR" workflow with `gh`.
- **`references/workspaces.md`** — multiple working copies, naming, stale-working-copy recovery, cleanup.
- **`references/operations-and-recovery.md`** — the operation log, `jj undo`, `jj op restore`, recovering abandoned/lost work.
- **`references/revsets.md`** — the revset query language (`@`, `::`, `trunk()`, `mine()`, `conflicts()`, …) used everywhere a `-r` is accepted.
