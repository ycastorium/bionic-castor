# bionic-castor

Skills-only plugin for Claude Code and Codex.

## Workflow

`brainstorming` → `generate-tasks` → `implementing-tasks`, with `tdd` and
`ponytail` applied while writing code and `ponytail-review` as the reviewer's
complexity lens.

- `brainstorming` — classify a request as spike, bounded, or architectural;
  architectural work ends in one `design.md` (spec half + architecture half)
- `generate-tasks` — break `design.md` into a commit-sized `tasks.md` with
  interfaces, waves, and global constraints
- `implementing-tasks` — drive `tasks.md` to completion: task briefs, report
  files, review packages, a capped fix loop, recorded rulings, final review
- `tdd` — test-driven development with red-green-refactor

## Ponytail

- `ponytail` — force the laziest solution that actually works
- `ponytail-review` — code review focused on over-engineering
- `ponytail-audit` — whole-repo over-engineering audit
- `ponytail-debt` — harvest `NOTE:` shortcut comments into a debt ledger
- `ponytail-help` — quick-reference card for ponytail modes

## Tools

- `obsidian` — read and write Obsidian notes as plain files
- `jujutsu` — drive the `jj` CLI without falling back on git muscle memory
- `ripgrep` — use `rg` correctly: escaping, file selection, and output shaping
- `gh-stack` — manage stacked pull requests with the `gh stack` CLI extension
- `elixir` — develop Elixir code using current language guidance and focused references

## Installation

### Claude Code

```bash
cc --plugin-dir /path/to/bionic-castor
```

Or add this directory as a marketplace/plugin source in Claude Code settings.

### Codex

Install this directory as a local Codex plugin, or add the bundled marketplace:

```bash
codex plugin marketplace add /path/to/bionic-castor/.agents/plugins
```

The marketplace entry points back to this repository root, so the skills stay in
one shared `skills/` directory for both Claude Code and Codex.
