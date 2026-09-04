# Designing the Architecture Half

Load this when the architectural path reaches step 5. The spec half is approved; this settles the **how**: which components exist, how they integrate, which technologies are used, how data flows.

## Explore the codebase first

- Use the codebase memory graph, ast-grep, or ripgrep to find the modules, patterns, and conventions the work will touch. Follow existing patterns; a design that fights the codebase's conventions is wrong even if elegant in isolation.
- Where existing code has problems that affect the work (a file grown too large, tangled responsibilities), include targeted improvements as part of the design. Do not propose unrelated refactoring.
- If the codebase contradicts the approved spec half, raise it with the user. Do not silently reinterpret; if the resolution changes a business decision, update the spec sections first.

## Propose 2-3 technical approaches

Different decompositions, technologies, integration styles, sync vs async, where state lives. Respect Constraints; a constraint is not up for redesign. Lead with your recommendation. All approaches, including rejected ones, go into section 8 with the specific reason each was rejected.

## Design for isolation

Break the system into units that each have one purpose, communicate through defined interfaces, and can be understood and tested independently. For each unit you should be able to answer: what does it do, how do you use it, what does it depend on? If someone cannot understand a unit without reading its internals, or cannot change the internals without breaking consumers, the boundary needs work. Smaller focused files are also easier for you to edit reliably.

## Sketch where ambiguity lives

Sketches remove the most common hand-off failure: an implementer inventing an interface shape or algorithm the design never intended.

- **Interfaces as signatures.** For each new or significantly changed unit, its public surface as real signatures in the repo's language: function heads, struct or type definitions, module callbacks, trait or behaviour declarations, using the codebase's naming conventions and existing types.
- **Pseudocode for non-obvious logic.** Algorithms, state transitions, tricky glue as language-flavored pseudocode: real syntax for structure, `# ...` or `// ...` for the obvious parts. Show branching, iteration, error paths without boilerplate.
- **Honest.** Real module paths, real types, the repo's error idiom (`{:ok, _}/{:error, _}` in Elixir, `Result` in Rust). A sketch referencing invented types is worse than none.
- **Short.** 5-30 lines. Longer means the unit is too big; fix the boundary, not the sketch.
- **Illustrative.** Say once in the document that sketches show intent and the implementer may adjust details as long as the interface contract and logic shape hold.
- **Not for the trivial.** CRUD pass-throughs, config plumbing, and code following an existing pattern verbatim get a pointer to the pattern, not pseudocode.

## Diagrams

Every diagram is paired with prose: the diagram shows the shape, the prose explains why. No diagram floats unexplained; no flow is prose-only.

| Flow | Mermaid type |
|------|--------------|
| Components and their relations | `flowchart` |
| Interactions between components or services | `sequenceDiagram` |
| Decision logic | `flowchart` |
| Lifecycles | `stateDiagram-v2` |
| Data models and schema changes | `erDiagram` |

## Sections 8-15

8. **Considered Approaches**: each approach, what made it attractive, why it was rejected.
9. **System Overview**: the chosen architecture at a glance as a `flowchart`. Name real modules and files where they exist.
10. **Components**: per new or significantly modified unit: purpose, interface as a signature sketch, dependencies, real paths and existing patterns to follow.
11. **Data & Flows**: how data moves and changes, using the diagram table above.
12. **Implementation Sketches**: pseudocode for the non-obvious parts, each naming its unit and paired with a sentence on why it is shaped that way. Omit the whole section when nothing is non-obvious.
13. **Technology Choices**: libraries, services, storage, protocols: chosen and why, and what was deliberately not adopted.
14. **Error Handling & Edge Cases**: what can go wrong and what the system does about it.
15. **Testing Strategy**: what gets unit tests, what needs integration coverage, what is verified manually.

## Scaling down

An architectural request that follows an existing pattern closely still gets these sections, but each may be a sentence or two: which files are touched, which pattern is followed, one small diagram if there is any flow at all. The gate that matters is that generate-tasks has a concrete document to decompose, not that a ritual was performed.
