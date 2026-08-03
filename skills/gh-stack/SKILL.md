---
name: gh-stack
description: Use when working with stacked pull requests via the official `gh stack` GitHub CLI extension (github/gh-stack). Triggers on "gh stack", "stacked PRs", "stack of PRs", "stacked pull requests", "split this into a stack", "gh stack init/add/submit/sync/rebase/merge/link", "cascading rebase", "merge the stack", or when a repo has a `.git/gh-stack` metadata file. Covers the stack mental model, the create→iterate→submit→sync→merge lifecycle, restructuring with modify/unstack, conflict recovery, linking PRs from other tools (e.g. Jujutsu), and exit codes.
---

# Stacked PRs with `gh stack`

`gh stack` is GitHub's official CLI extension for stacked pull requests (public preview). A **stack** is an ordered chain of branches where each branch builds on the one below it; each branch gets its own PR whose base is the branch beneath it, so reviewers see only that layer's diff. The extension automates the tedious parts: branch creation, cascading rebases, PR base wiring, and navigation.

```
frontend      → PR #3 (base: api-endpoints) ← top    (furthest from trunk)
api-endpoints → PR #2 (base: auth-layer)
auth-layer    → PR #1 (base: main)          ← bottom (closest to trunk)
─────────────
main (trunk)
```

`up` moves away from trunk, `down` moves toward it. Stack metadata lives in `.git/gh-stack` (local JSON, not committed); interrupted-rebase state in `.git/gh-stack-rebase-state`.

## Setup

```sh
gh extension install github/gh-stack   # requires gh v2.0+, authenticated (gh auth login)
```

Constraints to remember:
- All branches must live in the **same repository** — cross-fork stacks are unsupported.
- The feature is in public preview; the repo must have stacked PRs enabled (exit code 9 if not).
- `gh stack init` enables `git rerere` automatically so conflict resolutions are remembered across rebases.

## Golden rules for agents

- **Orient first.** Run `gh stack view --short` (or `--json` for parsing) before acting. Exit code 2 means you're not in a stack.
- **One logical layer per branch.** Each branch should be a reviewable unit; that's the whole point of stacking.
- **`gh stack add` only works from the top.** Check out the topmost branch (or `gh stack top`) before adding a layer.
- **Prefer `sync` for routine updates.** It fetches, reconciles the remote stack, fast-forwards trunk, cascade-rebases, force-pushes with lease, and syncs PR state — in one command. It's safe in automation: it only prompts on true divergence.
- **`push` ≠ `submit`.** `push` only pushes branches; `submit` pushes *and* creates/updates PRs and the stack object on GitHub.
- **After editing a lower layer, rebase upward.** Amend/commit on the lower branch, then `gh stack rebase` (or `--upstack` from that branch) so upper layers include the change, then `gh stack push`.
- **Never delete stack branches manually while stacked.** Use `modify` (drop) or `unstack` to restructure.
- **Non-interactive contexts (CI, agents):** use `submit --auto`, `merge --yes`, and know that `sync` aborts (successfully, without pushing) on divergence.

## Command reference

### Lifecycle: create → iterate → submit

```sh
gh stack init                      # interactive; offers current branch as first layer
gh stack init feat-a feat-b        # adopt existing / create missing branches, in order
gh stack init --base develop feat  # non-default trunk

# ...commit on the first branch...

gh stack add api-routes            # new branch at HEAD, on top of the stack (must be on top)
gh stack add -Am "Add login"       # stage all + commit + auto-named branch (e.g. 03-24-add_login)
gh stack add -um "Fix bug" layer2  # stage tracked-only + commit + explicit name (-A/-u exclusive, need -m)

gh stack view                      # full view: order, PR links, last commit (pager)
gh stack view --short              # branch names only
gh stack view --json               # machine-readable

gh stack submit                    # push all + open editor to draft PR titles/descriptions (Ctrl+S submits)
gh stack submit --auto             # skip editor; auto titles; new PRs created as drafts
gh stack submit --auto --open      # ...as ready-for-review instead
```

`submit` links the PRs into a Stack on GitHub. If every PR in the stack is already merged, `submit` starts a fresh stack rooted at trunk for the unmerged branches.

### Keeping in sync

```sh
gh stack sync             # fetch → reconcile remote stack → ff trunk → cascade rebase → push (lease) → sync PRs
gh stack sync --prune     # also delete local branches for merged PRs
gh stack push             # just push active branches (force-with-lease, not atomic; excludes merged/queued)
gh stack rebase           # fetch + cascading rebase, trunk upward; auto --onto over merged PRs
gh stack rebase --downstack   # only trunk → current branch
gh stack rebase --upstack     # only current branch → top
gh stack rebase --no-trunk    # no fetch; only restack branches onto each other
gh stack rebase --preserve-dates  # committer date = author date
```

**Divergence during `sync`** (local and remote stack compositions genuinely differ): interactively you choose — adopt the remote as source of truth, delete the stack object on GitHub (then `gh stack submit` recreates it from local), or cancel. Remote-ahead-only changes (PRs appended on GitHub) are pulled down automatically without prompting.

### Rebase conflicts

```sh
# rebase pauses and prints conflicted files with line numbers
# ...resolve, then:
git add <files>
gh stack rebase --continue
# or roll everything back:
gh stack rebase --abort
```

Exit code 3 = rebase conflict; 7 = rebase already in progress.

### Restructuring

```sh
gh stack modify           # interactive TUI: drop (x), fold down/up (d/u), insert (i/I),
                          # reorder (Shift+↑/↓), rename (r), undo (z); Ctrl+S applies all + cascading rebase
gh stack modify --continue   # after resolving an apply-phase conflict
gh stack modify --abort      # restore pre-modify state
```

`modify` preconditions: active stack, clean working tree, no rebase in progress, no PR queued for merge, linear history. Merged-PR branches can't be modified. After modifying an already-submitted stack, run `gh stack submit` to replace the stack on GitHub.

```sh
gh stack unstack          # (alias: delete) unstack on GitHub + remove local tracking
gh stack unstack 7        # by stack number, works without local checkout
gh stack unstack --local  # only drop local tracking, keep the GitHub stack
```

Merged/merging/queued PRs can't be removed from a GitHub stack. Big restructures: `unstack`, then `gh stack init <branches...>` (existing branches are adopted).

### Merging

```sh
gh stack merge            # interactive: pick how far up to merge, method, confirm
gh stack merge 42         # merge everything up to and including PR #42
gh stack merge 7          # merge a stack by stack number (purely remote)
gh stack merge --yes --squash   # non-interactive
```

All-or-nothing: if any selected PR can't merge, none do. PRs must be open and non-draft; branch protection is evaluated at merge time and **cannot be bypassed**. With a merge queue, PRs are enqueued together (method flags ignored) and may land in separate groups. When a bottom PR merges, GitHub cascade-rebases the remaining PRs so the next one targets trunk.

### Checkout & navigation

```sh
gh stack checkout             # interactive picker: local + remote stacks, searchable (/)
gh stack checkout 7           # by stack number (fetches + sets up locally if remote-only)
gh stack checkout 42          # by PR number; also accepts PR URLs
gh stack checkout feat-auth   # by branch name (local stacks only)

gh stack switch     # interactive branch picker within the current stack
gh stack up [n]     # toward the top (away from trunk); clamps at bounds
gh stack down [n]   # toward trunk
gh stack top / bottom / trunk
```

### Interop with other tools (Jujutsu, Sapling, git-town)

`gh stack link` creates/updates the stack **on GitHub only** — no local tracking. Ideal when branches are managed by jj or other tools locally:

```sh
gh stack link feat-a feat-b feat-c    # bottom → top; pushes branches, creates missing PRs,
                                      # fixes wrong PR bases, links into a stack (additive only)
gh stack link 10 20 30                # by PR numbers (or URLs)
gh stack link 7 48 feat-ui            # append to existing stack #7
gh stack link --base develop --open a b c
```

In a colocated jj repo: manage commits/bookmarks with jj, push bookmarks (or let `link` push), then `gh stack link` the bookmark names bottom-to-top.

### Utilities

```sh
gh stack alias        # install `gs` wrapper in ~/.local/bin (gs push == gh stack push)
gh stack alias gst    # custom name; --remove to uninstall
gh stack feedback "title"   # open a discussion on github/gh-stack
GH_STACK_THEME=light gh stack view   # force light/dark palette (auto default)
```

## Exit codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Generic error |
| 2 | Not in a stack / stack not found |
| 3 | Rebase conflict |
| 4 | GitHub API failure |
| 5 | Invalid arguments or flags |
| 6 | Disambiguation required (branch in multiple stacks) |
| 7 | Rebase already in progress |
| 8 | Stack locked by another process |
| 9 | Stacked PRs not enabled for this repository |
| 10 | Modify session interrupted; recovery required |

## Common workflows

**Start a stack from work in progress on `main`:**
```sh
gh stack init          # accept current branch or name the first layer
gh stack add -Am "second layer"
gh stack submit --auto
```

**Respond to review feedback on a middle layer:**
```sh
gh stack checkout <branch>     # or gh stack down/up to reach it
# ...edit, commit (or amend)...
gh stack rebase --upstack      # replay upper layers on the fix
gh stack push                  # update all PRs
```

**Trunk moved / bottom PR merged:**
```sh
gh stack sync --prune          # one command: rebase everything, push, clean up merged branches
```

**Land the whole stack:**
```sh
gh stack merge --yes --squash
gh stack sync --prune
```
