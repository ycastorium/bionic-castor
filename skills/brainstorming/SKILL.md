---
name: brainstorming
description: "Use before any creative work - creating features, building components, adding functionality, modifying behavior, or answering feasibility questions like 'can we...' or 'is it possible to...'. Applies even when the change looks trivial."
---

# Brainstorming

Turn an idea into an approved design before any code. The ceremony scales with the task; the approval gate never does. The architectural path produces one `design.md` that records the **what and why** (problem, goals, options, decisions) and the **how** (components, flows, technology), which the generate-tasks skill consumes.

<HARD-GATE>
Do NOT write code, scaffold a project, invoke an implementation skill, or take any implementation action until you have told the user what you intend and they have approved it. This holds on every path. A two-sentence design in chat still needs a yes.
</HARD-GATE>

## Three Paths

Before your first question, explore the project context (files, docs, recent commits), classify the request, and say the classification out loud with your reasoning so the user can override it: *"This looks bounded, so I'll present a short design here rather than write a doc. Sound right?"*

- **Spike**: a feasibility question ("can we...", "is it possible...", "quick and dirty is fine") whose output is an answer, not code you keep. Present the question and what you will try in 2-3 sentences, get a nod, investigate as cheaply as correctness allows, report a recommendation. No document. Anything built stays labeled throwaway; keeping it is a new request that gets its own classification.
- **Bounded**: a well-scoped change to a flow that already exists in this repo to read: a bug fix, a flag, a small endpoint, a one-file refactor. Understanding the kind of app is not enough; if there is no existing flow to change, it is not bounded. Ask the clarifying questions that matter (one per message), present a short design in chat, STOP until the user says yes. No document.
- **Architectural**: new projects, new subsystems, new capabilities, changes that restructure how components fit or alter interfaces others depend on, or anything with real options to weigh. Full process below, ending in `design.md`.

When in doubt, take the heavier path. The ratchet is one-way: hidden complexity discovered mid-task upgrades the path. Stop, say so, and step up. Nothing downgrades mid-task.

| Excuse | Reality |
|--------|---------|
| "Too simple to need a design" | Simple means a short design, not no design. Two sentences in chat, then approval. |
| "I'll call it bounded and skip the doc" | Reaching for a label to skip work IS the doubt. Take the heavier path. |
| "The design is obvious, I'll start while they read it" | The gate is the approval, not the design's length. Present, then stop. |
| "I understand this kind of app, so it's bounded" | Bounded measures the repo, not your familiarity. No existing flow means architectural. |
| "The spike works, so I'll keep the code" | A spike's output is an answer. Keeping the code is a new request. |
| "It grew, but I'm almost done" | Hidden complexity upgrades the path mid-task. Stop and say so. |
| "The user seems in a hurry" | Hurry is not a classification criterion. Hidden design decisions are where rushed work is wasted. |

## Spike Path

1. Explore enough context to frame the probe.
2. Present the question and probe plan in 2-3 sentences. Get a nod.
3. Investigate as cheaply as correctness allows.
4. Report findings as a recommendation. Label anything built as throwaway.

## Bounded Path

1. Explore project context.
2. Ask the clarifying questions that matter, one per message, multiple choice when possible.
3. Present a short design in chat: **what** changes, **why**, files touched, how you will test it, **done when** (the observable condition).
4. STOP and wait for an explicit yes.
5. Implement: load the **tdd** skill and the **ponytail** skill and proceed directly. No document, no task list.

## Architectural Path

Create a task for each item and complete them in order.

1. **Explore project context**: files, docs, recent commits. If the request spans multiple independent subsystems ("a platform with chat, billing, and analytics"), flag it before asking detailed questions and help the user decompose it; each sub-project gets its own design → tasks → implementation cycle. Brainstorm the first one.
2. **Ask clarifying questions**: one per message, multiple choice preferred. Understand purpose, constraints, success criteria. Stop when you can state what is being built and why without guessing.
3. **Propose 2-3 problem-level options**: scope variations, build vs buy vs adapt, phased vs all-at-once, which users or flows to serve first. Lead with your recommendation and reasoning. Stay at what-and-why altitude here; technical choices come in step 5.
4. **Present the spec half**: sections 1-7 of the document structure, each scaled to its complexity (a few sentences to 200-300 words). Ask after each section whether it looks right. Get approval of the whole spec half before designing the how.
5. **Design the architecture half**: read `references/architecture.md` in this skill directory, then explore the codebase, propose 2-3 technical approaches with your recommendation, and present sections 8-15 the same way, section by section with approval.
6. **Critique review**: attack the approved design as a skeptical colleague would. What would I do differently starting over? Which option was dismissed too quickly? What did I miss: unstated requirements, affected users, failure modes, migrations, concurrency, security, operations? Which assumptions were never validated with the user? Where does the design bend if an open question resolves the other way? Bring material findings back to the user before writing; otherwise say it came up clean. Either way the findings go into the document.
7. **Write `design.md`**: follow the Design Document Structure. Save to the project's Obsidian vault at `specs/YYYY-MM-DD-<topic>/design.md` (user preference overrides). If you do not know the vault location, load the **obsidian** skill.
8. **Self-review**: the checklist below. Fix inline, no re-review. For a large or high-stakes design, dispatch an independent reviewer with `design-document-reviewer-prompt.md` instead of relying on your own read.
9. **User review gate**: "Design written to `<path>`. Please review it and tell me if you want changes." Wait. Apply changes, re-run the self-review, repeat until approved.
10. **Hand off**: suggest loading the **generate-tasks** skill. It is the only next step; do not start implementing.

## Design Document Structure

Write for a reader with zero context. A new stakeholder reads the spec half and understands what is being built and why; a new developer reads the architecture half and could build it. Record reasoning, not just conclusions. Scale sections to the project: a small feature gets short sections, not fewer sections.

**Spec half** (what and why):

1. **Summary**: two or three sentences, what we are building and why it matters.
2. **Context & Problem**: what hurts today, what happens if we do nothing. Define domain terms a newcomer would not know.
3. **Goals & Non-Goals**: explicit lists.
4. **Considered Options**: every problem-level option discussed, including discarded ones: what it was, what made it attractive, the specific reason it was rejected. Prevents relitigating.
5. **Chosen Direction**: the option picked and why, described as outcomes and behavior a user or caller experiences.
6. **Success Criteria**: observable, checkable conditions.
7. **Constraints**: hard boundaries the design must respect: existing stack, compliance, deadlines, compatibility. Global rules an implementer must obey (version floors, naming rules, exact values) go here verbatim; generate-tasks copies them into every task list.

**Architecture half** (how), detailed in `references/architecture.md`:

8. **Considered Approaches**: every technical approach discussed, with rejection reasons.
9. **System Overview**: components and their relations, mermaid `flowchart`, real module and file names.
10. **Components**: for each new or changed unit: purpose, interface as a signature sketch in the repo's language, dependencies, existing pattern to follow.
11. **Data & Flows**: mermaid diagrams paired with prose.
12. **Implementation Sketches**: pseudocode for the non-obvious parts only. Omit the section when nothing is non-obvious.
13. **Technology Choices**: what was chosen, why, and what was deliberately not adopted.
14. **Error Handling & Edge Cases**.
15. **Testing Strategy**: what gets unit tests, integration coverage, manual verification.

**Closing sections:**

16. **Critique Findings**: what was reconsidered, what was missed then addressed, accepted limitations.
17. **Open Questions**: anything deferred and what would resolve it.

## Self-Review

1. **Placeholders**: any "TBD", "TODO", vague requirement, hand-waved mechanism ("somehow syncs")? Fix.
2. **Consistency**: goals match the chosen direction; diagrams match prose; a component's interface matches how other sections use it; sketches match the interfaces they claim.
3. **Coverage**: every Goal maps to a component or flow; nothing serves a Non-Goal; every Constraint is respected.
4. **Scope**: focused enough for one task list, or does it need decomposition?
5. **Ambiguity**: could any requirement be read two ways? Pick one and say it.
6. **Pointers are real**: referenced files, modules, patterns exist or are clearly marked new. Never invent paths.
7. **Sketches**: repo language, real types, codebase idioms, short, marked illustrative. Every non-obvious mechanism has one; nothing trivial does.
8. **Feasibility**: could generate-tasks decompose this into commit-sized tasks without guessing? Sharpen vague sections.
9. **YAGNI**: anything designed that no goal asks for? Cut it.
10. **Audience**: a junior developer could follow the reasoning without prior context; no unexplained jargon.
