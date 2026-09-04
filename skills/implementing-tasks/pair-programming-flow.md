# Pair-Programming Flow

You are the **implementer**. You write the code with the user; only review is delegated. Work directly in the base workspace, one task at a time in list order; `## Execution Waves` is ignored.

## What differs from the shared loop in SKILL.md

- **Brief**: still generate it (`scripts/task-brief`) and read it; it keeps you on the task's exact criteria and interfaces, and the reviewer needs the same file.
- **Implement**: load the **tdd** skill (red, watch it fail, green, refactor) and the **ponytail** skill. Pair with the user: surface decisions instead of silently choosing. The user is present, so ask freely, but still write each answer to `## Rulings` in `tasks.md` so it survives the session.
- **Report file**: none. Tell the reviewer so, and put your RED/GREEN evidence (commands and output) in the dispatch prompt in its place, under 15 lines.
- **Review**: `scripts/review-package` over BASE..HEAD, then `task-reviewer-prompt.md`.
- **Fix loop**: you fix the findings yourself, amend them into the task commit, and run the scoped re-review (`re-review-prompt.md`) with the fix diff and the covering test output. Same five-round cap; at the cap, adjudicate with the user and record rulings.
- **Commit, mark done**: as in SKILL.md.

When all tasks are checked off, return to SKILL.md: Final Review, then Once All Tasks Are Done.
