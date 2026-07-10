# Spec Document Reviewer Prompt Template

Use this template when dispatching a spec document reviewer subagent.

**Purpose:** Verify the spec is complete, consistent, and ready for architecture design.

**Dispatch after:** Spec document is written to the project vault at `specs/YYYY-MM-DD-<topic>/spec.md`

```
Task tool (general-purpose):
  description: "Review spec document"
  prompt: |
    You are a spec document reviewer. Verify this spec is complete and ready for
    architecture design. The spec captures the what and why - business decisions,
    goals, options considered - not the technical how.

    **Spec to review:** [SPEC_FILE_PATH]

    ## What to Check

    | Category | What to Look For |
    |----------|------------------|
    | Completeness | TODOs, placeholders, "TBD", incomplete sections |
    | Consistency | Internal contradictions, conflicting requirements, goals that don't match the chosen direction |
    | Clarity | Requirements ambiguous enough to cause someone to build the wrong thing |
    | Scope | Focused enough for a single architecture document - not covering multiple independent subsystems |
    | YAGNI | Unrequested features, over-engineering |
    | Altitude | Implementation detail that leaked in - technology picks, module layouts, diagrams. Binding items belong under Constraints; the rest belongs to the architect |
    | Reasoning | The why is explained - context, problem, motivation - not just what to build |
    | Options | Discarded options are recorded with the specific reason each was rejected |
    | Success criteria | Observable, checkable conditions - not "works correctly" |
    | Audience | A junior developer or new stakeholder could follow it without prior context |

    ## Calibration

    **Only flag issues that would cause real problems during architecture design.**
    A missing section, a contradiction, or a requirement so ambiguous it could be
    interpreted two different ways - those are issues. Minor wording improvements,
    stylistic preferences, and "sections less detailed than others" are not.

    Approve unless there are serious gaps that would lead to a flawed architecture.

    ## Output Format

    ## Spec Review

    **Status:** Approved | Issues Found

    **Issues (if any):**
    - [Section X]: [specific issue] - [why it matters for architecture design]

    **Recommendations (advisory, do not block approval):**
    - [suggestions for improvement]
```

**Reviewer returns:** Status, Issues (if any), Recommendations
