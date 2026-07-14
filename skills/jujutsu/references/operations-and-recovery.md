# The operation log, undo, and recovery

jj records **every repository-modifying command** as an entry in the operation log. This makes recovery in jj far simpler and safer than git's reflog: you can undo a single command or rewind the entire repository to any past state, atomically. When anything looks wrong, reach here before improvising.

## Seeing what happened

```
jj op log                    # list operations, newest first, each with an op ID and description
jj op log -p                 # include the diff of what each operation changed
jj op show <op-id>           # details of one operation
```

Each entry shows an operation ID (hex prefix), the command that ran, the time, and a summary. `jj log` shows commits; `jj op log` shows the *history of your actions* on those commits.

## Undo

```
jj undo                      # reverse the most recent operation
```

`jj undo` puts the repo back the way it was before the last command. Undo is itself an operation, so running `jj undo` again re-applies it (you can toggle). `jj undo` only targets the *latest* operation — to rewind to a specific earlier point, use `jj op restore <op-id>` (below) rather than trying to undo a mid-history operation.

## Restore (rewind the whole repo)

```
jj op restore <op-id>        # reset the ENTIRE repo state to how it was at <op-id>
```

`op restore` is the "time machine": it makes the working copy, all commits, and all bookmarks exactly what they were at that operation. Use it when several bad commands piled up and you want to jump cleanly back to a known-good point, rather than undoing one at a time.

## Common recovery scenarios

**"I abandoned/squashed the wrong commit."**
```
jj undo                      # if it was the last thing you did
# otherwise find it:
jj op log                    # locate the operation just before the mistake
jj op restore <that-op-id>
```

**"A rebase went sideways / conflicts everywhere."**
```
jj op restore <op-before-the-rebase>   # rewind, then redo the rebase more carefully
```

**"I lost a commit — it's not in `jj log` anymore."**
Abandoned and rewritten commits are not gone; they're just not shown by the default log revset. Find them:
```
jj op log -p                 # scan operations to spot the change ID / hash
jj log -r 'all()'            # show everything the repo still knows about
# Once you have the change ID, resurrect it:
jj new <change>              # or `jj rebase -r <change> -d <dest>` to graft it back
```

**"jj says the working copy is stale."** (multi-workspace)
```
jj workspace update-stale
```

## Why this is safe to lean on

- Undo/restore are **operations themselves**, so you can undo an undo. You can never paint yourself into a corner.
- Rewritten commits, abandoned changes, and old bookmark positions remain reachable via the op log until jj garbage-collects — which does not happen automatically in normal use.
- This is why the golden rule is *"when in doubt, `jj undo`"* rather than manually reconstructing state: the safe path is always available.

## Housekeeping (rarely needed)

```
jj op log                    # inspect before pruning
# Old operations and unreachable commits are cleaned up by jj's maintenance; you
# almost never need to run gc manually. If disk is a concern, consult `jj util gc`
# in the installed version's help — but do NOT gc if you might still need to
# recover recent abandoned work.
```
