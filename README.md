# bionic-castor

Skills-only plugin for Claude Code and Codex. Bundles the following skills,
copied verbatim:

- `ponytail` — force the laziest solution that actually works
- `ponytail-audit` — whole-repo over-engineering audit
- `ponytail-help` — quick-reference card for ponytail modes
- `ponytail-review` — code review focused on over-engineering
- `ponytail-debt` — harvest `ponytail:` comments into a debt ledger
- `implementing-tasks` — drive a tasks.md list through implementation
- `tdd` — test-driven development with red-green-refactor
- `generate-tasks` — break an architecture doc into a commit-sized task list
- `brainstorming` — explore intent, requirements and business decisions into a spec
- `architect` — turn an approved spec into a technical architecture document
- `obsidian-cli` — read and write Obsidian notes via the `obsidian` CLI

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
