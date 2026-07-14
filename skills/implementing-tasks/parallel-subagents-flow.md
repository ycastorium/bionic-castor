# Parallel Subagent Flow

You (this session) are the **parent/orchestrator**. You do not write task code yourself. Tasks run wave by wave, following the `## Execution Waves` section of `tasks.md`: every task in a wave is implemented concurrently in its own throwaway git worktree (or jj workspace), then merged back into the base workspace (the feature branch or worktree from Branching Mode) before the next wave starts.

All worktree operations go through **worktrunk** (`wt`) — it places worktrees per project config, runs the project's post-create hooks (env setup, deps), and its `wt merge` collapses the rebase/fast-forward/remove dance into one command. Do not fall back to raw `git worktree` unless `wt` is unavailable.

**On a Jujutsu repo**, skip worktrunk entirely: use `jj workspace add` for each task's isolated workspace and `jj rebase` + bookmark advance for reintegration, in place of `wt switch --create` / `wt merge`. Load the [[jujutsu]] skill (`references/workspaces.md` and `references/git-interop.md`) before this flow. The per-wave procedure below gives the jj command alongside each git/worktrunk one.

## Requirements

- `tasks.md` must have a `## Execution Waves` section. If it doesn't (older list), derive the waves yourself — two tasks share a wave only when there is no dependency path between them **and** their `Create:`/`Modify:` pointers touch disjoint files — and confirm the grouping with the user via `AskUserQuestion` before starting.
- Waves are a hard barrier: wave N+1 must not start until every wave-N task is merged into the base and the full test suite passes there.

## Flow

```mermaid
flowchart TD
    Wave([Next wave in tasks.md]) --> Size{More than one task?}
    Size -->|no| Direct[Run the task in the base,<br>exactly like the sequential flow]
    Direct --> Gate
    Size -->|yes| Fan[wt switch --create per task / jj workspace add,<br>branched off the base tip]
    Fan --> Impl[Dispatch implementers concurrently,<br>one per worktree/workspace]
    Impl --> Rev[Review and commit each task<br>inside its own worktree/workspace]
    Rev --> Merge[wt merge / jj rebase + bookmark set,<br>one task at a time, test after each -<br>removes the worktree/workspace itself]
    Merge --> Gate{Full suite green in base?}
    Gate -->|no| Fix[Parent fixes the base<br>before continuing]
    Fix --> Gate
    Gate -->|yes| More{More waves?}
    More -->|yes| Wave
    More -->|no| Done([All tasks done - back to SKILL.md])
```

## Per-wave procedure

**Single-task wave:** skip the worktree machinery — run the task directly in the base workspace, exactly as the sequential flow does (implementer subagent, review agent, commit, mark done).

**Multi-task wave:**

1. Mark every task of the wave `in_progress` in the native task list.
2. For each task, create a throwaway worktree branched from the current tip of the base (`--no-cd` keeps your shell in the base; `-y` skips prompts):

   ```
   wt switch --create task/<n>-<slug> --base <base-branch> --no-cd -y
   ```

   Get each worktree's path from `wt list --format json`.

   **On Jujutsu**, create a workspace instead, then reposition its `@` onto the base bookmark's tip (a fresh workspace otherwise starts empty on top of the current workspace's parent, not the base):

   ```
   jj workspace add ../task-<n>-<slug>
   cd ../task-<n>-<slug> && jj new <base-bookmark>
   ```
3. Dispatch one implementer subagent per task, all concurrently, each pointed at its own worktree path (or jj workspace path). Pick or create agents the same way the sequential flow does: prefer a specialised agent over a generic one, choose the model by task complexity. Every implementer follows the same rules: read `architecture.md` and its task first, **tdd skill mandatory**, **ponytail** for simplicity, implement only the assigned task, and **ping the parent and wait** when blocked — never guess.
4. Each task is code-reviewed inside its own worktree/workspace (separate review agent, **ponytail-review** plus the task's acceptance criteria) and committed there as **a single commit** with the task's suggested message (`git commit` on git — review fixes get amended in, not stacked as extra commits; `jj commit -m` on Jujutsu — review fixes go back into `@` with `jj squash` before the final `jj describe`/`jj commit`, so it stays one change).

**Reintegration (parent does this, one task at a time):**

1. Merge each task branch into the base with worktrunk, pointing `-C` at the task worktree. `--no-squash` preserves the task's commit message (the branch already holds exactly one commit); the command rebases onto the base, fast-forwards it, and removes the worktree in one go:

   ```
   wt -C <task-worktree-path> merge --no-squash <base-branch>
   ```

   Always pass the base branch explicitly — `wt merge` defaults to the repo's default branch, which is not where wave commits land.

   **On Jujutsu**, rebase the task's change onto the current tip of the base bookmark, then advance the bookmark to the new tip:

   ```
   jj rebase -s <task-change-id> -d <base-bookmark>
   jj bookmark set <base-bookmark> -r <task-change-id>
   ```

   Do this one task at a time, in order — each task rebases onto wherever the base bookmark landed after the previous task's merge, the same "one at a time" serialization as `wt merge`.
2. After each merge, run the tests in the base. If the rebase stops on a conflict, resolve it in the task worktree/workspace and re-run the merge (jj: `jj resolve`, then re-run the rebase); test failures after a clean merge are fixed in the base. Either way it is the parent's job — do not re-dispatch the implementer for integration problems.
3. Only after its merge is in and the base is green: mark the task `completed` in the native list and check its box plus acceptance criteria in `tasks.md` in Obsidian.

**Wave gate:** with all wave tasks merged, run the whole test suite in the base once more. Green → next wave. Red → fix the base before proceeding; never start a wave on a broken base.

## Handling confusion

Same rule as the sequential flow: a blocked agent pings the parent and waits; the parent answers from `architecture.md`/`tasks.md` or escalates to the user via `AskUserQuestion` and relays the answer. With several agents in flight, answer pings promptly — a blocked agent stalls only its own worktree/workspace, but the wave cannot close until every task lands.

## Cleanup

Before declaring a wave (or the run) done: `wt list` shows no leftover task worktrees, `git branch` shows no leftover `task/` branches, and every commit is reachable from the base. `wt merge` removes the worktree and merged branch itself; if a task was abandoned mid-wave, clean it up with `wt remove -f -D task/<n>-<slug>` so nothing dangles.

**On Jujutsu**, `jj workspace list` should show no leftover task workspaces once every task's change has been rebased into the base. `jj rebase -s` above moves the commit but does not remove the workspace record — run `jj workspace forget <name>` after each successful merge, then delete the now-empty workspace directory. If a task was abandoned mid-wave, `jj workspace forget <name>` and `jj abandon <task-change-id>` before removing the directory.
