# jj workspaces

A **workspace** is an additional working copy backed by the same repository. It is jj's equivalent of `git worktree`, with one key twist: **each workspace has its own working-copy commit `@`**, so you can have different in-progress changes checked out in each directory simultaneously — no stashing, no branch switching.

Use workspaces when you need to work on two things at once: e.g. keep a long-running change open while making a quick fix on trunk, or run tests in one workspace while editing in another.

## Creating and listing

```
jj workspace add <path>                 # create a new workspace at <path>
jj workspace add --name fix ../myrepo-fix   # give it an explicit name
jj workspace list                       # show all workspaces and their @ commits
jj workspace root                       # print the root of the CURRENT workspace
```

`jj workspace add <path>` creates a new directory with its own working copy. The new workspace's `@` starts as a fresh empty commit on top of the current workspace's `@` parent (you can `jj new <somewhere>` inside it to reposition). All workspaces share the same commits, bookmarks, and operation log — they are views into one repo, not clones.

Each workspace has a name (the default one is usually `default`). The name appears in `jj workspace list` and disambiguates working-copy commits in `jj log` (shown as `<name>@`).

## The stale-working-copy gotcha

Because all workspaces share one repo, an operation in workspace A (a rebase, an abandon, an amend) can rewrite the commit that workspace B has checked out. When you return to B, jj notices its recorded `@` no longer matches the repo and reports a **stale working copy**.

Fix it with:

```
jj workspace update-stale
```

This updates the stale workspace's working copy to the current version of its commit. It does **not** discard your uncommitted edits — those were already snapshotted into that workspace's `@` on the last command run there. If you get "stale working copy" errors, run `update-stale` rather than trying to reset by hand.

To avoid surprises: prefer to keep each workspace focused on its own stack of commits, so operations in one don't rewrite the commit another has checked out.

## Cleaning up

```
jj workspace forget <name>              # stop tracking a workspace (its @ commit remains in the repo)
```

`jj workspace forget` removes jj's record of the workspace; it does not delete the directory. After forgetting, delete the directory yourself if you no longer need it. The commits that were in that workspace are still reachable in the repo (and via the op log) — forgetting a workspace does not lose committed work.

## Workspaces vs. bookmarks vs. `jj new`

- Need a second working copy on disk to edit two things at once → **workspace**.
- Just need to move to a different line of work in the same directory → `jj new <change>` or `jj edit <change>`. No workspace required; jj has no "dirty tree" blocking the switch.
- Need a named, pushable pointer → **bookmark** (independent of workspaces).

Reach for a workspace only when you genuinely need concurrent working copies; otherwise `jj new`/`jj edit` is lighter.
