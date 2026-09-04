---
name: ponytail-help
description: >
  Quick-reference card for all ponytail modes, skills, and commands.
  One-shot display, not a persistent mode. Trigger: /ponytail-help,
  "ponytail help", "what ponytail commands", "how do I use ponytail".
---

# Ponytail Help

Display this reference card when invoked. One-shot, do NOT change mode or
persist anything.

## Levels

| Level | Trigger | What changes |
|-------|---------|-------------|
| **Lite** | `/ponytail lite` | Build what's asked, name the lazier alternative in one line. |
| **Full** | `/ponytail` | The ladder enforced: YAGNI → reuse → stdlib → native → one line → minimum. Default. |
| **Ultra** | `/ponytail ultra` | YAGNI extremist. Deletion before addition. Challenges requirements before building. |

Level sticks until changed or session end.

## Skills

| Skill | Trigger | What it does |
|-------|---------|--------------|
| **ponytail** | `/ponytail` | Lazy mode itself. Simplest solution that works. |
| **ponytail-review** | `/ponytail-review` | Over-engineering review of a diff: `L42: yagni: factory, one product. Inline.` |
| **ponytail-audit** | `/ponytail-audit` | Same hunt over the whole repo: ranked list of what to delete or replace. |
| **ponytail-debt** | `/ponytail-debt` | Harvest `NOTE:` shortcut comments into a debt ledger. |
| **ponytail-help** | `/ponytail-help` | This card. |

Codex uses `@ponytail` style; Claude Code uses the slash forms above.

## Where it plugs in

- **brainstorming** bounded path hands off to tdd + ponytail.
- **implementing-tasks**: implementers run under ponytail; the task reviewer applies ponytail-review as its complexity lens after acceptance and correctness.

## Deactivate

"stop ponytail", "normal mode", or `/ponytail off`. Resume with `/ponytail`.
