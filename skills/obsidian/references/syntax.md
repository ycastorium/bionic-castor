# Obsidian-flavored Markdown syntax

A note is a `.md` file. Everything below is written as plain text into that file with `Write`/`Edit`. This is what distinguishes an Obsidian note from generic Markdown.

## Frontmatter (properties)

A YAML block fenced by `---` at the very top of the file (must be the first line). Keys become Obsidian "properties".

```markdown
---
title: My Note
aliases: [MN, "Meeting Notes"]
tags: [project-x, meeting]
status: active
priority: 3
done: false
created: 2026-07-14
due: 2026-07-20T15:00
cssclasses: [wide]
---
```

Type is inferred from YAML shape, and Obsidian renders each accordingly:

| Property type | YAML form | Example |
|---|---|---|
| Text | scalar string | `status: active` |
| List | inline `[a, b]` or block `- a` | `tags: [a, b]` |
| Number | bare number | `priority: 3` |
| Checkbox | boolean | `done: true` |
| Date | `YYYY-MM-DD` | `created: 2026-07-14` |
| Date & time | `YYYY-MM-DDTHH:MM` | `due: 2026-07-20T15:00` |

Reserved keys Obsidian treats specially: `tags`, `aliases`, `cssclasses`. Tags in frontmatter are written without the `#`.

## Internal links (wikilinks)

| Syntax | Meaning |
|---|---|
| `[[Note Name]]` | Link to `Note Name.md` (filename without extension, any folder) |
| `[[Note Name\|Label]]` | Link with custom display text |
| `[[Note Name#Heading]]` | Link to a specific heading within a note |
| `[[Note Name#^blockid]]` | Link to a specific block (see block references) |
| `[[#Heading]]` | Link to a heading in the same note |
| `[[folder/Note Name]]` | Path-qualified link when names collide across folders |

Obsidian resolves links by filename, not path, so names should be unique unless path-qualified. External links use standard Markdown `[text](https://url)`.

## Embeds (transclusion)

Prefix any link with `!` to embed the target's content inline instead of linking to it:

| Syntax | Embeds |
|---|---|
| `![[Note Name]]` | The whole note |
| `![[Note Name#Heading]]` | A section under a heading |
| `![[Note Name#^blockid]]` | A single block |
| `![[image.png]]` | An image (also `![[diagram.pdf]]`, audio, video) |
| `![[image.png\|300]]` | Image resized to 300px wide |

## Block references

Append `^anchor` at the end of a line/paragraph to give that block a stable ID other notes can link or embed:

```markdown
This is an important conclusion. ^key-finding
```

Reference it with `[[Note#^key-finding]]` or embed with `![[Note#^key-finding]]`.

## Tags

Inline in the body: `#tag`, `#nested/tag`. Tags allow letters, digits, `_`, `-`, `/`; they cannot be purely numeric and contain no spaces. The same tag can also live in the `tags:` frontmatter list (without the `#`). Both forms are indexed by Obsidian's search and tag pane.

## Tasks (checkboxes)

```markdown
- [ ] Open task
- [x] Completed task
- [/] In progress (custom status)
- [-] Cancelled (custom status)
```

`[ ]` and `[x]` are standard. Any single character between the brackets is a valid "status" that themes/plugins (e.g. Tasks) may style. The Tasks plugin also reads inline metadata on the task line:

```markdown
- [ ] Ship release 📅 2026-07-20 ⏫ #release
```

(`📅` due date, `⏫` priority, etc. — only if the Tasks plugin is installed.)

## Callouts

Blockquotes with a `[!type]` marker on the first line:

```markdown
> [!note] Optional title
> Body of the callout.

> [!warning]- Collapsed by default
> The trailing `-` starts it folded; `+` starts it expanded.
```

Common types: `note`, `info`, `tip`, `success`, `question`, `warning`, `failure`, `danger`, `bug`, `example`, `quote`, `abstract`, `todo`.

## Headings, and how links target them

Standard `#`/`##`/`###` Markdown headings. Heading text is what `[[Note#Heading]]` targets, so keep headings stable — renaming a heading breaks links pointing at it.

## Comments

`%% this text is hidden in reading view %%` — inline or multi-line. Never rendered, useful for notes-to-self.

## Math and diagrams

- Inline math `$e^{i\pi}$`, block math with `$$ ... $$` (KaTeX).
- Mermaid diagrams in a ` ```mermaid ` fenced block.

## What NOT to do when editing files directly

- Don't put content inside the frontmatter `---` fences unless it's valid YAML.
- Don't rename/move a note without updating the `[[wikilinks]]` that point at it — the app auto-updates these, direct file edits do not.
- Don't assume a tag or link resolves; verify with `Glob`/`Grep` against the vault.
