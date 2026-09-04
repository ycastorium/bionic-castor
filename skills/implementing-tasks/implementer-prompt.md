# Implementer Prompt Template

Fill and dispatch for every subagent-implemented task. Everything you paste here, and everything the agent prints back, stays in your context for the rest of the session, so hand artifacts over as file paths.

```
Subagent (prefer a language/domain-specialised type, else general-purpose):
  description: "Implement Task N: [task name]"
  model: [MODEL — REQUIRED, per SKILL.md Model Selection]
  prompt: |
    You are implementing Task N: [task name].

    ## Requirements

    Read your task brief first: [BRIEF_FILE]
    It is your requirements: the Global Constraints that bind every task,
    and the task's code pointers, interfaces, acceptance criteria, and
    commit message. Use exact values from it verbatim.

    Work from: [WORKDIR]
    Version control: [git | jj — commit with `jj commit -m`, no staging]

    ## Context

    [One line on where this task fits. Interfaces and rulings from earlier
    tasks the brief cannot know, exact signatures. Nothing else: no session
    history, no summaries of previous tasks.]

    ## Before You Begin

    If the brief is ambiguous about requirements, approach, or dependencies,
    ask now with status NEEDS_CONTEXT. Do not guess.

    ## Your Job

    1. Load the **tdd** skill and follow it: one failing test, watch it
       fail for the right reason, minimal code to pass, refactor. The
       interface and behaviours come from the brief, so skip tdd's
       "confirm with user" planning items.
    2. Load the **ponytail** skill: the laziest solution that actually
       works. Reuse what the codebase has before writing anything new.
    3. Implement only this task. Respect the code pointers; follow the
       Reference: patterns.
    4. Run the focused test while iterating; run the full suite once before
       committing. Output must be pristine: no stray warnings.
    5. Commit with the brief's suggested message.
    6. Self-review your own diff: every acceptance criterion met, nothing
       extra built, names match what things do, tests assert real
       behaviour not mocks.
    7. Report.

    ## You Do Not Dispatch Subagents

    Do all of this yourself. Never spawn a helper, and never spawn a
    reviewer to check your work. Review is the orchestrator's job and is
    already scheduled after your report; one you spawn duplicates it at
    full cost and its approval counts for nothing.

    ## When You Are Stuck

    Stop and say so. Bad work is worse than no work; you will not be
    penalised for escalating. Escalate when the task needs an architectural
    decision with several valid answers, when you cannot find clarity in
    the code, when the task needs restructuring the brief did not
    anticipate, or when you have been reading file after file without
    progress. Report BLOCKED with what you are stuck on, what you tried,
    and what help you need.

    ## After Review Findings

    If the review finds issues you will be resumed with them. Fix them,
    re-run the tests covering the amended code, and append a fix report to
    your report file: what changed, the covering tests, the command, the
    output. Reviewers do not re-run tests; your report is the evidence.
    Amend the fixes into the task's commit (jj: `jj squash`) so the task
    stays one commit. Reply with the same short status contract.

    ## Report

    Write the full report to [REPORT_FILE]:
    - What you implemented (or attempted, if blocked)
    - TDD evidence: RED command + failing output and why the failure was
      expected; GREEN command + passing output
    - Files changed
    - Self-review findings
    - Concerns

    Then reply with ONLY, under 15 lines:
    - **Status:** DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
    - Commit (short id + subject)
    - One-line test summary ("14/14 passing, output pristine")
    - Concerns, if any
    - The report file path

    If BLOCKED or NEEDS_CONTEXT, put the specifics in the reply itself.
    Never silently produce work you are unsure about.
```

**Placeholders:** `[MODEL]`, `[BRIEF_FILE]` (from `scripts/task-brief`), `[WORKDIR]` (base workspace, or the task worktree in parallel mode), `[REPORT_FILE]` (`<artifacts>/task-N-report.md`).
