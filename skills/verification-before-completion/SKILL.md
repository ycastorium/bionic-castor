---
name: verification-before-completion
description: Use when about to say work is done, fixed, passing, or working; before committing, opening a PR, marking a task complete, or relaying a subagent's success report; and whenever a sentence you are about to write implies success.
---

# Verification Before Completion

A claim about the state of the code is a claim about command output you have read in this turn. No fresh output, no claim.

## The Gate

Before any status claim:

1. **Identify** the command that proves it.
2. **Run** it, in full, now. A run from earlier in the session proved the tree it ran on, not this one.
3. **Read** the output: exit code, failure count, warnings.
4. **Claim** what the output shows, with the evidence: "34/34 pass", "exit 0", "2 failures in `auth_test`". If it shows less than you hoped, say that instead.

## What Proves What

| Claim | Needs | Not enough |
|-------|-------|------------|
| Tests pass | Test command output, 0 failures | An earlier run, "should pass" |
| Linter clean | Linter output, 0 errors | A partial run, extrapolation |
| Build succeeds | Build command, exit 0 | Linter or tests passing |
| Bug fixed | The original symptom, reproduced, now gone | Code changed, assumed fixed |
| Regression test works | It failed before the fix and passes after | It passes once |
| Agent finished the task | `git diff` / `jj diff` shows the change, and you checked it | The agent said "done" |
| Requirements met | Each requirement checked against the code, one by one | Tests pass |

For a regression test written after the fix: revert the fix, run (must fail), restore, run (must pass).

## Scope It to the Claim

Run what proves the claim, not more. A one-module change claimed as "tests pass for X" needs X's tests; a claim that the branch is ready to merge needs the full suite. Say which one you ran.

## Red Flags

These mean you are about to claim without evidence. Stop and run the command.

- "should", "probably", "seems to", "looks correct"
- "Done!", "Fixed!", "All good" written before the output was read
- About to commit, push, or open a PR
- Relaying a subagent's report as fact
- "The linter passed, so it builds"

| Excuse | Reality |
|--------|---------|
| "It passed a minute ago" | Something changed since, or you would not be claiming again. Run it. |
| "I'm confident in the change" | Confidence is not output. |
| "The agent said it's done" | Agent reports are claims. Check the diff. |
| "It's a one-line change" | One-line changes break builds too. The command is cheap. |
| "Running the suite is slow" | Then run the scoped tests and say the claim is scoped. |
