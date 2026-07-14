# Revsets: selecting commits

A **revset** is a query that resolves to a set of commits. Anywhere a jj command takes `-r`/`--revisions` (or `-s`, `-d`, `-b`), you can pass a revset expression, not just a single ID. Learning a handful of these makes `jj log`, `jj rebase`, and cleanup dramatically more precise.

Quote revsets in the shell when they contain spaces or special characters: `jj log -r 'main..@'`.

## Symbols and single commits

| Expr | Meaning |
|------|---------|
| `@` | the working-copy commit |
| `@-` | parent of `@` |
| `@+` | child(ren) of `@` |
| `<change>` | a commit by change ID (shortest unique prefix works) |
| `<bookmark>` | the commit a bookmark points to (e.g. `main`) |
| `<bookmark>@origin` | a remote-tracking bookmark (e.g. `main@origin`) |
| `root()` | the virtual root commit |

Ancestry operators (suffix/prefix `-` and `+` can chain: `@--` is grandparent):

## Sets and ranges

| Expr | Meaning |
|------|---------|
| `x-` / `x+` | parents / children of `x` |
| `::x` | `x` and all its ancestors |
| `x::` | `x` and all its descendants |
| `x::y` | the DAG range: descendants of `x` that are ancestors of `y` |
| `x..y` | ancestors of `y` that are NOT ancestors of `x` (git's `x..y`) |
| `x | y` | union |
| `x & y` | intersection |
| `x ~ y` | difference (in `x`, not in `y`) |
| `~x` | complement (everything except `x`) |

## Functions (most useful for agents)

| Function | Returns |
|----------|---------|
| `all()` | every commit (careful — large) |
| `mine()` | commits authored by the current user |
| `trunk()` | the main-line commit (resolves to `main`/`master`/default branch) |
| `heads(x)` | commits in `x` with no children in `x` (the tips) |
| `roots(x)` | commits in `x` with no parents in `x` |
| `empty()` | commits that make no file changes |
| `merges()` | merge commits |
| `conflicts()` | commits containing unresolved conflicts |
| `description(pat)` | commits whose message matches `pat` |
| `author(pat)` / `author_date(...)` | filter by author / date |
| `bookmarks()` | commits pointed to by any local bookmark |
| `remote_bookmarks()` | commits pointed to by remote bookmarks |
| `tags()` | tagged commits |
| `files(pat)` | commits that touch files matching `pat` |
| `latest(x, n)` | the `n` most recent commits in `x` (default 1) |
| `immutable_heads()` | the heads of the immutable set (rewrite guard boundary) |

## Practical examples

```
# The default-ish view: your stack plus trunk
jj log -r 'trunk()::@ | @'

# Everything you authored that isn't yet on trunk
jj log -r 'mine() ~ ::trunk()'

# Just the commits between the remote main and your working copy
jj log -r 'main@origin..@'

# All commits currently carrying conflicts
jj log -r 'conflicts()'

# Empty (no-op) commits you might want to abandon
jj log -r 'empty() & mine() ~ @'

# Rebase your whole stack (everything from the first commit after trunk up to @)
jj rebase -s 'roots(trunk()..@)' -d main@origin

# Abandon every empty descendant of a change
jj abandon -r 'empty() & <change>::'
```

## Tips

- When a command wants "this commit and everything built on it", use `x::`. When it wants "just this one", use `x`.
- `trunk()` is preferred over hard-coding `main`/`master` — it adapts to the repo.
- To discover what a revset resolves to before acting, run `jj log -r '<expr>'` first.
- Revset **aliases** (including `immutable_heads()`) are configurable in jj config; a repo may have customized them.
