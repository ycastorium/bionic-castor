# Design Document Reviewer Prompt Template

Dispatch after `design.md` is written, for large or high-stakes designs, instead of relying on the author's self-review. Specify the model explicitly (a mid-tier model is enough; an omitted model inherits the session's most expensive one).

```
Subagent (general-purpose):
  description: "Review design document"
  model: [MODEL]
  prompt: |
    You are a design document reviewer. Verify this design is complete,
    internally consistent, and ready to be broken into commit-sized tasks.
    The spec half (sections 1-7) records what and why; the architecture
    half (8-15) records how.

    **Design to review:** [DESIGN_FILE_PATH]
    **Repository root:** [REPO_ROOT] (verify referenced paths exist here)

    ## What to Check

    | Category | What to Look For |
    |----------|------------------|
    | Completeness | TODOs, placeholders, "TBD", hand-waved mechanisms ("somehow syncs") |
    | Consistency | Goals vs chosen direction; diagrams vs prose; a component's interface vs how other sections use it; sketches vs the interfaces they implement |
    | Coverage | Every Goal maps to a component or flow; nothing serves a Non-Goal; every Constraint is respected |
    | Clarity | Requirements ambiguous enough that someone could build the wrong thing |
    | Scope | Focused enough for a single task list, not several independent subsystems |
    | Pointers | Referenced files, modules, and patterns exist in the repo or are clearly marked new |
    | Sketches | Repo language, real types and idioms, short, illustrative; present for non-obvious logic, absent for trivial code |
    | Diagrams | Every flow has a diagram paired with prose |
    | Feasibility | Concrete enough to decompose into commit-sized tasks without guessing |
    | Error handling | Failure modes named with a stated response |
    | Testing | A strategy that would catch the design breaking |
    | YAGNI | Components, layers, or flexibility no goal asks for |
    | Reasoning | Discarded options and approaches recorded with the specific reason each was rejected |
    | Audience | A junior developer or new stakeholder could follow it without prior context |

    ## Calibration

    Only flag issues that would cause real problems during task generation or
    implementation: a goal with no design, a violated constraint, an invented
    path, a contradiction, a mechanism too vague to plan from. Wording,
    style, and "this section is shorter than that one" are not issues.

    Approve unless there are gaps that would lead to a flawed task list.

    ## Output Format

    Begin directly with the status. No preamble.

    **Status:** Approved | Issues Found

    **Issues (if any):**
    - [Section X]: [specific issue] - [why it matters downstream]

    **Recommendations (advisory, do not block approval):**
    - [suggestions]
```

**Reviewer returns:** Status, Issues, Recommendations
