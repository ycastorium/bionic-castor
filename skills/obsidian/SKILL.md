---
name: obsidian
description: This skill should be used when the user asks to "save this to obsidian", "write a note in obsidian", "search my vault", "check obsidian for X", "append to today's daily note", "add a tag/property to this note", "list backlinks", "find orphan notes", "move/rename/delete a note in obsidian", or otherwise wants to read from or write to an Obsidian vault.
---

# Obsidian

An Obsidian vault is just a folder of plain Markdown (`.md`) files. There is no database and no required app — a note is a text file with an optional YAML frontmatter block and Obsidian-flavored Markdown in the body. So the ordinary file tools are all that's needed: `Read`, `Write`, `Edit`, `Glob`, and `Grep`. Do not depend on any `obsidian` CLI — it drives the live desktop app and fails often.

## Finding the vault

If you don't know where the vault is, ask the user for its location, then save it to your memory (write a `reference`-type memory file with the vault's absolute path and add a line to `MEMORY.md`) so you don't have to ask again. A vault's root always contains a `.obsidian/` directory — you can confirm a path is a vault by checking for it.

Once you have the vault root, treat every path in this skill as relative to it.

## Core operations (all via file tools)

### Read a note

Notes are addressed by their relative path. Wikilinks reference a note by its filename without the `.md` extension (`[[Meeting Notes]]` → `Meeting Notes.md`), which may live in any folder, so resolve a wikilink to a file with `Glob` when the folder is unknown:

```
Glob: **/Meeting Notes.md
```

Then `Read` the resulting path.

### Create a note

`Write` the file. A typical note starts with a frontmatter block, then the body:

```markdown
---
tags: [meeting, project-x]
status: active
created: 2026-07-14
---

# Meeting Notes

Agenda:
```

Frontmatter is optional — plenty of notes are body-only. Place the file wherever the user asks; if unspecified, ask or drop it in the vault root. Don't overwrite an existing note unless the user asked to replace it — `Read` first to check.

### Update a note

Use `Edit` to change existing content, add a property, or insert a line. To append or prepend, `Read` the file and `Edit` at the end or start of the body (keep new content below the frontmatter block, never inside it).

### Frontmatter properties

Properties are keys in the YAML frontmatter block at the very top of the file, delimited by `---` lines. To set or change one, `Edit` the YAML. Types are conveyed by YAML shape — `tags: [a, b]` is a list, `priority: 3` a number, `done: true` a checkbox, `due: 2026-07-20` a date. See `references/syntax.md` for the property conventions Obsidian recognizes.

### Search the vault

Use `Grep` over the vault root. Prefer `Grep` with `-C` context and `output_mode=content` when the goal is to quote or locate specific lines; use `output_mode=files_with_matches` to just find which notes mention something.

```
Grep: pattern="quarterly review", path=<vault root>, output_mode=content, -C=2
```

### Tags

Tags appear two ways: inline in the body as `#tag`, and in a `tags:` frontmatter property. To find all notes with a tag, `Grep` for both forms (`#meeting` and a `tags:` line containing `meeting`). To add a tag, either add `#tag` in the body or add it to the `tags:` list in frontmatter.

### Tasks

Tasks are checkbox list items: `- [ ]` (todo) and `- [x]` (done). Find them with `Grep` (`^\s*- \[ \]` for open tasks) and toggle one by `Edit`ing `[ ]` ↔ `[x]` on that line.

### Daily notes

A daily note is an ordinary note named for the date, usually `YYYY-MM-DD.md`, often under a `Daily/` or `Journal/` folder (the exact folder/format is a vault setting — check `.obsidian/daily-notes.json` if present, or ask). To append to today's, resolve today's date, `Glob`/`Read` the file (create it if missing), and `Edit` in the new content.

### Links and graph health

Links are `[[wikilinks]]` in note bodies. For maintenance requests, use `Grep`:

- **Backlinks** to a note: `Grep` the vault for `[[<note name>` (also matches `[[note|alias]]` and `[[note#heading]]`).
- **Broken links**: extract `[[targets]]` and check each resolves to a file via `Glob`.
- **Orphans / dead-ends**: notes with no incoming / no outgoing `[[links]]`.

### Move / rename / delete

Moving or renaming a note breaks every `[[wikilink]]` and embed pointing at it (Obsidian's app normally auto-updates these; editing files directly does not). So after a move/rename, `Grep` for the old name and `Edit` the referencing notes. Use `git mv`/`mv` to move, and confirm with the user before deleting — a plain file delete is not recoverable through Obsidian's trash.

## Syntax reference

Obsidian-flavored Markdown — wikilinks, embeds, block references, callouts, frontmatter property conventions, and tasks — is documented in `references/syntax.md`. Consult it before writing note bodies so the output renders correctly in Obsidian rather than as plain Markdown.
