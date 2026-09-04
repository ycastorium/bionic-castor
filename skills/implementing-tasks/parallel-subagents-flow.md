# Parallel Subagent Flow

You are the **orchestrator**. Tasks run wave by wave per `## Execution Waves`: every task in a wave is implemented concurrently in its own throwaway worktree (git, via worktrunk `wt`) or jj workspace, then merged back into the base before the next wave starts. Waves are a hard barrier.

Worktree operations go through **worktrunk**: `wt merge` collapses rebase, fast-forward, and removal into one command. Fall back to raw `git worktree` only if `wt` is unavailable. On Jujutsu skip worktrunk entirely; load the **jujutsu** skill (`references/workspaces.md`, `references/git-interop.md`) first.

## Per wave

**Single-task wave**: run it in the base exactly as the sequential flow does.

**Multi-task wave:**

1. Mark every task of the wave in progress.
2. Create one workspace per task, branched from the base tip, under `.workspaces/`:

   ```
   wt switch --create task/<n>-<slug> --base <base-branch> --no-cd -y
   ```
   Path from `wt list --format json`. If `wt` won't honour `.workspaces/`, use `git worktree add .workspaces/task-<n>-<slug>`.

   Jujutsu (a fresh workspace starts on the current workspace's parent, so reposition it):
   ```
   jj workspace add .workspaces/task-<n>-<slug>
   cd .workspaces/task-<n>-<slug> && jj new <base-bookmark>
   ```
   Warm build caches from the base per the table in SKILL.md, skipping what worktrunk hooks already populated.
3. Record **BASE per workspace** (`git rev-parse HEAD` inside it, or the jj change id of `@-`).
4. Generate each task's brief, then dispatch all implementers in the same response so they run concurrently, each with `implementer-prompt.md`, its own `[WORKDIR]`, its own report path, and an explicit model. Same context rule as the sequential flow: one line of placement, consumed interfaces, relevant rulings, nothing else.
5. Each task is reviewed inside its own workspace: run `scripts/review-package` **from that workspace** (the range lives in its history), dispatch `task-reviewer-prompt.md`, run the fix loop there. Each task ends as **one commit** with its suggested message; review fixes are amended in (jj: `jj squash`).
6. Answer pings promptly. A blocked agent stalls only its workspace, but the wave cannot close until every task lands.

## Reintegration (you, one task at a time)

```
wt -C <task-worktree-path> merge --no-squash <base-branch>
```
`--no-squash` keeps the task's commit message; the command rebases onto the base, fast-forwards, and removes the worktree. Always pass the base branch: `wt merge` defaults to the repo's default branch.

Jujutsu:
```
jj rebase -s <task-change-id> -d <base-bookmark>
jj bookmark set <base-bookmark> -r <task-change-id>
jj workspace forget task-<n>-<slug> && rm -rf .workspaces/task-<n>-<slug>
```

After each merge run the tests in the base. A rebase conflict is resolved in the task workspace and the merge re-run (jj: `jj resolve`, re-run the rebase); a test failure after a clean merge is fixed in the base. Both are your job; do not re-dispatch the implementer for integration problems. Only once the merge is in and the base is green: mark the task done in the native list and `tasks.md`.

**Wave gate**: all wave tasks merged → full suite in the base once more. Green → next wave. Red → fix the base first.

## Cleanup

Before closing a wave: `wt list` shows no task worktrees, no `task/` branches remain, every commit is reachable from the base. Abandoned task: `wt remove -f -D task/<n>-<slug>`. Jujutsu: `jj workspace list` is clean; abandoned task: `jj workspace forget` then `jj abandon <change-id>`, then delete the directory.

When all waves are done, return to SKILL.md: Final Review, then Once All Tasks Are Done.
