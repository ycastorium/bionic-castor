# jj ↔ git interop, colocated repos, and making PRs

This is the reference for everything that crosses the jj/git boundary: colocated repos, pushing and fetching, how bookmarks map to branches, and the full "open a PR" workflow. Read this before pushing.

## Two ways jj lives with git

jj always stores its commits in a git backend, but there are two layouts:

- **Non-colocated (default clone):** `jj git clone <url>` creates a `.jj/` directory with an internal git repo hidden inside it. There is no top-level `.git/`. Git-only tools won't see the repo; use `jj` for everything.
- **Colocated:** both `.jj/` and `.git/` sit at the repo root. jj and git share the same object store and the same commits. `git log`, `git status`, IDEs, and git hooks all work. jj **automatically imports** git refs and **automatically exports** jj bookmarks on essentially every command, so the two views stay in sync.

Create/convert:

```
jj git clone --colocate <url> [dir]   # clone as colocated
jj git init --colocate                # turn the CURRENT git repo into a colocated jj repo
```

### Working in a colocated repo

- **Prefer `jj` for anything that writes history** (commits, rebases, bookmark moves). Auto-export then updates the git branches to match.
- **`git` is fine for reads** (`git log`, `git show`, `git diff`, `git blame`) and for tools/hooks that expect git.
- If you *do* run a git write (e.g. a script does `git commit`), jj will import it as a new commit on the next jj command — you'll see it appear as an operation. This is safe but muddies history; avoid interleaving writes when you can.
- After external git changes that jj hasn't picked up, force a sync with `jj git import`. To force jj's bookmarks out to git refs, `jj git export`. In colocated repos these usually run automatically, so you rarely call them by hand.

## Bookmarks ↔ branches

- A jj **bookmark** exports to a git **branch** of the same name.
- A remote branch `origin/main` appears in jj as the remote-tracking bookmark `main@origin`; your local bookmark is just `main`.
- `jj bookmark list --all` shows local and remote bookmarks and whether they're tracked.
- Track a remote branch so pulls update your local bookmark: `jj bookmark track main@origin`.
- Because bookmarks don't auto-advance, the branch you push is wherever you last *set* the bookmark — not necessarily `@`.

## Fetching and updating

```
jj git fetch                       # fetch from the default remote (updates *@origin bookmarks)
jj git fetch --remote origin
jj git fetch --branch 'glob:*'     # limit which branches to fetch

# Rebase your work onto freshly fetched trunk:
jj rebase -d main@origin           # or -d main if your local bookmark tracks it
```

There is no `jj pull`. Fetch, then rebase (or `jj new`) onto the updated remote bookmark. jj auto-rebases your descendants, so a stack moves as one.

## Pushing

The rule to remember: **push moves a bookmark's commit to the remote. Set the bookmark first.**

```
# Push a specific bookmark you've positioned at @:
jj bookmark set feature-x -r @
jj git push --bookmark feature-x         # -b is the short form

# Push ALL bookmarks that changed:
jj git push --all                        # every bookmark
jj git push --tracked                    # only bookmarks that track a remote

# Push a change WITHOUT manually naming a bookmark:
jj git push --change @                    # -c @ : auto-creates a bookmark named
#   push-<change-id> pointing at @, then pushes it. Handy for quick PR branches.

# Delete a remote branch:
jj bookmark delete feature-x
jj git push --bookmark feature-x          # pushes the deletion
```

Guards jj enforces on push (and how to satisfy them):

- **Won't push a commit with no description** → run `jj describe -m "..."` first.
- **Won't push a commit with unresolved conflicts** → `jj resolve` first.
- **Won't push the working-copy commit as-is if it's empty** → put your work in it, or push the parent.
- **Won't move a bookmark backward / non-fast-forward on the remote unintentionally** → it will tell you; use `jj git push --bookmark x` after reconciling, or `--allow-backwards` on the bookmark move if that's genuinely intended.

## End-to-end: open a pull request

Assuming a colocated repo (or any repo with a GitHub remote) and the `gh` CLI:

```
# 1. Sync with the remote trunk
jj git fetch
jj rebase -d main@origin            # if you started before the latest trunk

# 2. Make sure every commit you're shipping has a real description
jj log                              # eyeball the stack; no "(no description)" entries
jj describe <change> -m "..."       # fix any that are missing

# 3. Put a bookmark on the tip of your work
jj bookmark create my-feature -r @  # or `jj bookmark set` if it already exists

# 4. Push the bookmark (creates origin/my-feature)
jj git push --bookmark my-feature

# 5. Open the PR with gh (branch name == bookmark name)
gh pr create --head my-feature --base main --fill
```

Updating the PR after review:

```
# Edit files (they snapshot into @), or amend a specific commit:
jj edit <change> && <make edits> && jj new @-   # amend a mid-stack commit
# or just work on @ and squash down as needed

jj bookmark set my-feature -r @     # advance the bookmark to the new tip
jj git push --bookmark my-feature   # force-updates the remote branch (jj handles the non-ff)
```

Because jj rewrites commits (new hashes) on every amend/rebase, PR updates are effectively force-pushes — this is normal and expected in a jj workflow. jj computes the safe update for you; you don't pass `--force`.

## Quick reference

| Task | Command |
|------|---------|
| Clone colocated | `jj git clone --colocate <url>` |
| Convert git repo | `jj git init --colocate` |
| Fetch | `jj git fetch` |
| Force-sync from git refs | `jj git import` |
| Force jj bookmarks to git | `jj git export` |
| Track remote branch | `jj bookmark track main@origin` |
| Push one bookmark | `jj git push -b <name>` |
| Push a change (auto-bookmark) | `jj git push -c @` |
| Push everything tracked | `jj git push --tracked` |
