# Scoped Re-Review Prompt Template

Dispatch after each fix round. The re-reviewer verdicts each finding and inspects the fix diff for new breakage, nothing else. The full review already happened.

```
Subagent (general-purpose):
  description: "Re-review Task N fix round R"
  model: [MODEL — REQUIRED; cheap-to-mid for small fix diffs]
  prompt: |
    You are re-reviewing one fix round. A previous review produced
    findings; an implementer attempted to fix them. Verdict each finding
    and inspect the fix diff. Read-only: do not touch the working tree,
    index, HEAD, or bookmarks. Do not dispatch subagents.

    ## The Task
    Read the brief: [BRIEF_FILE]

    ## Findings Under Verification
    [FINDINGS — copied verbatim from the previous review, one per bullet]

    ## The Fix
    Implementer's report, fix reports appended at the end: [REPORT_FILE]
    Fix base: [FIX_BASE] (the head the previous review saw)  Head: [HEAD]
    Diff file: [DIFF_FILE]

    Read the diff file once; do not re-run git or jj. The implementer
    re-ran the covering tests and appended the output; confirm the fix
    report names them and shows output, and verify the claims against
    the diff. Do not re-run the suite; a focused test only if the code
    raises a specific doubt.

    ## Scope
    Verdict every finding. Inspect the fix diff for problems the fix
    itself introduced. Do NOT re-review code the fix did not touch; an
    issue entirely outside the fix diff goes under Out-of-Scope and does
    not block this round.

    ## Output
    Begin with the first finding's verdict. No preamble.

    ### Finding Verdicts
    - **[finding one-liner]** — ADDRESSED | NOT ADDRESSED, file:line
      evidence. "Attempted" is not addressed; the defect must be gone.

    ### New Breakage in the Fix Diff
    Severity + file:line, or "None".

    ### Out-of-Scope Observations
    Non-blocking, or "None".

    ### Verdict
    **Fix round:** All findings addressed, no new Critical/Important |
    Findings remain open: [list]
```

**Placeholders:** `[MODEL]`, `[BRIEF_FILE]`, `[FINDINGS]`, `[REPORT_FILE]` (omit in Pair Programming mode), `[FIX_BASE]`, `[HEAD]`, `[DIFF_FILE]` (from `scripts/review-package <tasks.md> FIX_BASE HEAD`).
