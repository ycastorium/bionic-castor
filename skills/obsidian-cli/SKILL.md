---
name: obsidian-cli
description: This skill should be used when the user asks to "save this to obsidian", "write a note in obsidian", "search my vault", "check obsidian for X", "append to today's daily note", "add a tag/property to this note", "list backlinks", "find orphan notes", "move/rename/delete a note in obsidian", or otherwise wants to read from or write to an Obsidian vault via the `obsidian` CLI.
---

# Obsidian CLI

The `obsidian` binary drives a running Obsidian desktop app to read and write vault content from the command line. It is not a standalone Markdown-file tool — it talks to the live app instance, so changes appear immediately in the UI and respect whatever plugins/sync are configured.

## Prerequisite

Obsidian.app must already be running with the target vault open. If a command returns empty output or an error mentioning no active vault, ask the user to open Obsidian first rather than guessing at a fix.

## Never run bare `obsidian`

Running `obsidian` with no arguments does NOT print usage — it tries to launch a new Obsidian UI instance and can hang the shell indefinitely. Always pass a command. When unsure which command or option to use, run `obsidian --help` for the command list or `obsidian help <command>` for one command's options — never experiment with a bare invocation.

## Invocation shape

```
obsidian <command> [key=value ...] [flag ...] [vault=<name>]
```

- Options are `key=value`; flags (like `total`, `verbose`, `permanent`) take no value.
- Quote values containing spaces: `name="My Note"`.
- Use `\n` / `\t` inside `content=` values for newlines/tabs.
- `file=` resolves like a wikilink (by name, no folder needed, extension optional). `path=` is the exact vault-relative path (`folder/note.md`). Prefer `file=` unless there's a name collision across folders.
- Most read/edit commands default to the currently active file in the Obsidian UI when both `file=` and `path=` are omitted.
- Pass `vault=<name>` to target a non-default vault when several are open.

## Core workflows

### Create and edit notes

```
obsidian create name="Meeting Notes" content="# Meeting Notes\n\nAgenda:\n"
obsidian append file="Meeting Notes" content="- decided X"
obsidian prepend file="Meeting Notes" content="tags: #meeting\n"
obsidian read file="Meeting Notes"
```

`create` fails if the file exists unless `overwrite` is passed. Use `append`/`prepend` for existing notes instead of `create overwrite`.

### Frontmatter properties

Properties are the YAML frontmatter block, not arbitrary text:

```
obsidian property:set file="Meeting Notes" name=status value=done
obsidian property:set file="Meeting Notes" name=priority value=3 type=number
obsidian property:read file="Meeting Notes" name=status
obsidian property:remove file="Meeting Notes" name=status
obsidian properties file="Meeting Notes"          # all properties on one file
obsidian properties total sort=count counts        # property usage across the vault
```

`type=` matters for how Obsidian renders the field in the UI (`text|list|number|checkbox|date|datetime`); omit it for plain text.

### Tags

```
obsidian tags file="Meeting Notes"
obsidian tags total sort=count counts               # most-used tags vault-wide
obsidian tag name=meeting verbose                    # every file using #meeting
```

Tags live inline in the note body (`#tag`) or in a `tags:` property — the CLI reads both.

### Tasks

```
obsidian tasks file="Meeting Notes" todo             # incomplete checkboxes in one file
obsidian tasks todo verbose                           # incomplete tasks, grouped by file with line numbers
obsidian task ref="Meeting Notes.md:12" toggle
obsidian task file="Meeting Notes" line=12 done
```

Run `tasks ... verbose` first to get `path:line` references, then target individual tasks with `task ref=`.

### Daily note

```
obsidian daily:path
obsidian daily:read
obsidian daily:append content="- followed up with client"
obsidian daily:prepend content="## Standup\n"
```

These operate on today's daily note without needing to know its filename or date format.

### Search

```
obsidian search query="quarterly review" limit=10
obsidian search:context query="quarterly review"     # includes matching lines, not just filenames
```

`search` returns matching files; `search:context` also returns the matching line(s) — prefer `search:context` when the goal is to quote or locate specific content, not just to know which notes mention something.

### Links and graph health

```
obsidian links file="Meeting Notes"        # outgoing links from this note
obsidian backlinks file="Meeting Notes" counts
obsidian unresolved total                   # links pointing at notes that don't exist
obsidian orphans total                      # notes nothing links to
obsidian deadends total                     # notes that link to nothing
```

Useful for vault maintenance requests ("find broken links", "what's not connected to anything").

### Listing and inspecting

```
obsidian files folder="Projects" ext=md
obsidian folders folder="Projects"
obsidian file file="Meeting Notes"          # size, created/modified timestamps
obsidian folder path="Projects" info=size
obsidian outline file="Meeting Notes" format=md
obsidian wordcount file="Meeting Notes" words
```

### Deleting and moving

```
obsidian move file="Meeting Notes" to="Archive/Meeting Notes.md"
obsidian rename file="Meeting Notes" name="2026-07-01 Meeting Notes"
obsidian delete file="Meeting Notes"            # goes to trash
obsidian delete file="Meeting Notes" permanent  # skips trash
```

Treat `delete ... permanent` as irreversible — confirm with the user before running it on anything they didn't explicitly ask to be permanently removed.

## Output formats

Many list commands accept `format=json|tsv|csv` (default is usually `tsv` or a plain table). Request `format=json` when the output needs to be parsed or filtered programmatically instead of just read.

## Everything else

Vault/plugin/theme management, sync, version history, bookmarks, templates, Bases queries, and developer/debug commands (`dev:*`, `eval`) are documented in `references/commands.md`. Consult it (or `obsidian help <command>`) for anything not covered above rather than guessing at flag names — the CLI has no tolerance for unknown options and fails silently or with a generic error.
