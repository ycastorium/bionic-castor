# Sequential Subagent Flow

You are the **orchestrator**. You never write task code; you dispatch subagents and keep the native task list and `tasks.md` current. Tasks run strictly one at a time, in list order, in the base workspace; `## Execution Waves` is ignored.

## What differs from the shared loop in SKILL.md

- **Dispatch**: one implementer per task with `implementer-prompt.md`, model chosen per Model Selection and stated explicitly. Record its agent identity; fix-loop rounds 1-3 resume it. Never run two implementers at once in this flow.
- **Context you add**: one line on where the task fits, exact `Produces` signatures from earlier tasks it consumes, and any `## Rulings` line that touches its area. Nothing else; a real session's dispatch hit 42k characters of pasted history.
- **Waiting**: while an agent runs, do local work (update the list, prepare the next brief). When idle, wait in bounded stretches and post one line of status; chase any agent that finished without reporting.
- **Pings**: answer from the brief and `design.md`, or rule and record. Never leave an agent waiting.
- **Review, fix loop, commit, mark done**: as in SKILL.md. You commit only if the implementer did not; normally the implementer commits and you verify the commit exists before marking done.

When all tasks are checked off, return to SKILL.md: Final Review, then Once All Tasks Are Done.
