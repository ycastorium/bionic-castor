---
name: tdd
description: Use when implementing any feature, bug fix, or behaviour change, before writing implementation code; also when the user mentions TDD, red-green-refactor, or test-first.
---

# Test-Driven Development

## Philosophy

**Core principle**: Tests should verify behavior through public interfaces, not implementation details. Code can change entirely; tests shouldn't.

**Good tests** are integration-style: they exercise real code paths through public APIs. They describe _what_ the system does, not _how_ it does it. A good test reads like a specification - "user can checkout with valid cart" tells you exactly what capability exists. These tests survive refactors because they don't care about internal structure.

**Bad tests** are coupled to implementation. They mock internal collaborators, test private methods, or verify through external means (like querying a database directly instead of using the interface). The warning sign: your test breaks when you refactor, but behavior hasn't changed. If you rename an internal function and tests fail, those tests were testing implementation, not behavior.

**Tests are code.** Every test must be read, maintained, and fixed when it breaks. A test that cannot catch a plausible bug is pure cost. The goal is the smallest set of tests that would catch the bugs you actually expect, not coverage.

See [tests.md](tests.md) for examples and [mocking.md](mocking.md) for mocking guidelines.

## What Deserves a Test

Before each test, name the bug it catches. If you cannot describe a realistic production mistake that would make it fail, do not write it.

**Test** (one test per behavior, not per input):

- Branching logic: each distinct outcome of a decision, not each input that reaches it
- Calculations and transformations where the wrong answer looks plausible
- Boundaries callers depend on: empty, one, many; limits; the error the caller must handle
- Contracts at trust boundaries: validation, auth, money, data writes
- Every bug fix: one regression test that reproduces it; same for anything that has bitten this codebase before

**Skip**:

- Pass-through code: getters, setters, constructors, delegation, thin wrappers over a library
- Constants, config values, enum members, type shapes; the compiler or type system already covers these
- Framework or library behavior (routing, ORM queries, serialization) unless you configured it in a non-obvious way
- Logging, metrics, and debug output
- Private helpers; they are covered through the public behavior that uses them
- Exhaustive input permutations when one representative case per outcome proves the logic
- Error paths that can only happen through programmer error (wrong type passed, impossible state), unless the language cannot catch them
- Anything you cannot name a plausible bug for

**Sizing rule**: the number of tests tracks the number of decisions in the code, not its line count. A 200-line function with two branches needs about two tests. Twenty near-identical tests that differ only by input are one parametrized test, or one test with the representative case plus the boundary that matters.

**Stopping rule**: stop when the next test would not change the implementation and would not catch a bug you believe is plausible. Passing tests that never drove a line of code are the signal you went too far.

## Anti-Pattern: Horizontal Slices

**DO NOT write all tests first, then all implementation.** This is "horizontal slicing" - treating RED as "write all tests" and GREEN as "write all code."

This produces **crap tests**:

- Tests written in bulk test _imagined_ behavior, not _actual_ behavior
- You end up testing the _shape_ of things (data structures, function signatures) rather than user-facing behavior
- Tests become insensitive to real changes - they pass when behavior breaks, fail when behavior is fine
- You outrun your headlights, committing to test structure before understanding the implementation

**Correct approach**: Vertical slices via tracer bullets. One test → one implementation → repeat. Each test responds to what you learned from the previous cycle. Because you just wrote the code, you know exactly what behavior matters and how to verify it.

```
WRONG (horizontal):
  RED:   test1, test2, test3, test4, test5
  GREEN: impl1, impl2, impl3, impl4, impl5

RIGHT (vertical):
  RED→GREEN: test1→impl1
  RED→GREEN: test2→impl2
  RED→GREEN: test3→impl3
  ...
```

## Workflow

### 1. Planning

When exploring the codebase, use the project's domain glossary so that test names and interface vocabulary match the project's language, and respect ADRs in the area you're touching.

Before writing any code:

- [ ] Confirm what interface changes are needed
- [ ] Confirm which behaviors to test (prioritize)
- [ ] Identify opportunities for [deep modules](deep-modules.md) (small interface, deep implementation)
- [ ] Design interfaces for [testability](interface-design.md)
- [ ] List the behaviors to test (not implementation steps)
- [ ] Get approval on the plan

**Where the answers come from depends on how you were invoked.** Working from a task brief (implementing-tasks) or an approved in-chat design (brainstorming bounded path): the interface is the brief's `Produces` signature or the design, the behaviors are the acceptance criteria, and approval already happened; do not ask the user again. Working ad hoc: ask, "What should the public interface look like? Which behaviors are most important to test?"

**Budget the list.** Apply [What Deserves a Test](#what-deserves-a-test) to the behavior list before starting: cut anything you cannot name a plausible bug for, and merge input variations into one representative case per outcome. Aim for the shortest list that would catch the bugs you expect; a feature with three decisions usually needs three to five tests, not fifteen.

### 2. Tracer Bullet

Write ONE test that confirms ONE thing about the system:

```
RED:   Write test for first behavior → run it → it fails for the right reason
GREEN: Write minimal code to pass → run it → it passes, output pristine
```

This is your tracer bullet - proves the path works end-to-end.

**Watch it fail.** Run the test before writing production code and read the failure. It must fail because the behavior is missing, not because of a typo or a missing import. A test that passes immediately is testing existing behavior; fix the test. If you did not watch it fail, you do not know it can catch the bug.

**Name the break before writing the body.** What production change would make this test fail, and is that change a bug or a decision? Cannot name one → redesign around an observable behavior. Only a deliberate decision fails it (a constant's value, exact wording, private structure) → that is a change detector; test the behavior that depends on the decision instead. Derive the expected value by hand, as a literal or hand-checked fixture; an expectation computed by the code under test passes no matter what the code does.

### 3. Incremental Loop

For each remaining behavior:

```
RED:   Write next test → fails
GREEN: Minimal code to pass → passes
```

Rules:

- One test at a time
- Only enough code to pass current test
- Don't anticipate future tests
- Keep tests focused on observable behavior
- Apply the stopping rule each cycle: if the next test goes GREEN without a code change, it is testing something already covered; delete it rather than keep it

### Keep Test Code Simple

Test code is held to the same laziness as production code. Over-engineered tests are harder to trust than no tests.

- Inline setup. Three lines of repeated arrange code in three tests is fine; a builder, factory, or fixture abstraction is not justified until the same setup appears in three or more tests and is longer than a few lines
- No test base classes, test helper libraries, or custom assertion DSLs. Use the framework's assertions and the project's existing helpers
- A test reads top to bottom with no indirection: arrange, act, assert. If a reader must open another file to understand what the test checks, it is too abstract
- Prefer one test with a literal expected value over a test that reconstructs the expected value with logic
- Do not test the test helpers
- One behavior per test, but several assertions on the same result are fine; do not split one scenario into five tests to satisfy "one assertion per test"

### Reducing Churn

A test that breaks when behavior did not change is pinned to something that moves more often than the behavior. Pin tests to what is stable:

- **Test at the seam that survives refactors.** Usually the module or use-case boundary, not each function. If renaming or moving a function would require updating tests, the tests sit one level too low
- **Assert the minimum that proves the behavior.** Assert the fields you care about, not deep equality on the whole object; adding a field should not break unrelated tests. Match error types, not message text. Do not assert order unless order is the behavior. Avoid snapshot tests except for literal rendering contracts
- **Shared fixtures provide a valid baseline, not the scenario.** A shared app instance, test DB, or default valid entity is good; it keeps setup out of every test. The churn comes when a test asserts on a value that lives in the fixture. Every value a test asserts on is set explicitly in that test, so a fixture change breaks only tests whose behavior actually depends on it
- **Organize tests by behavior, not by source file.** A one-to-one `foo` to `foo_test` mapping ties tests to file layout, so moving code moves tests
- **Hand-written fakes over mock expectations.** `expect(x).called_with(...)` encodes the call graph. An in-memory fake at the boundary encodes only the contract and survives internal rewiring
- **When a refactor breaks a test without changing behavior, do not patch the assertion.** Delete the test (it was a change detector) or re-aim it at a higher level. A test patched twice gets rewritten
- **Delete tests with their behavior.** When a feature changes, tests "fixed" to keep passing usually stop testing anything

### 4. Refactor

After all tests pass, look for [refactor candidates](refactoring.md):

- [ ] Extract duplication
- [ ] Deepen modules (move complexity behind simple interfaces)
- [ ] Apply SOLID principles where natural
- [ ] Consider what new code reveals about existing code
- [ ] Run tests after each refactor step

**Never refactor while RED.** Get to GREEN first.

## Checklist Per Cycle

```
[ ] Test describes behavior, not implementation
[ ] Test uses public interface only
[ ] Test would survive internal refactor
[ ] I can name the production change that fails it
[ ] Watched it fail for the expected reason before writing code
[ ] Code is minimal for this test
[ ] No speculative features added
[ ] Test setup is inline; no new helper or fixture abstraction without three real uses
[ ] This test is not a duplicate of an existing one with a different input
[ ] Every value this test asserts on is set in this test, not inherited from a fixture
[ ] Output pristine: no errors or warnings
```

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Too simple to test" | If it has a decision in it, it is not too simple. If it is pass-through, skip it on purpose, not by accident. |
| "I'll write the tests after" | Tests written after pass immediately, which proves nothing. You never watched them fail, so you never proved they catch the bug. |
| "I already manually tested it" | No record, no re-run, forgotten cases. Automated tests run the same way every time. |
| "I'll write all the tests first, then all the code" | Horizontal slicing. Each test must respond to what the previous cycle taught you. |
| "Deleting X hours of code is wasteful" | Sunk cost. Keeping code you cannot trust is the waste. Rewrite from the tests. |
| "Hard to test, so I'll mock it" | Hard to test means hard to use. Fix the interface, see [mocking.md](mocking.md). |
| "Existing code has no tests" | You are improving it. Add a test for the behavior you touch. |

## Over-Testing Rationalizations

The opposite failure is just as real. These do not justify a test:

| Excuse | Reality |
|--------|---------|
| "More tests means safer" | More tests means more to read and fix. Safety comes from tests that catch plausible bugs, not from count. |
| "We should hit full coverage" | Coverage measures lines executed, not bugs caught. A pass-through line at 100% coverage is still untested in any useful sense. |
| "Let me cover every edge case" | Cover each outcome once plus the boundaries callers hit. Ten inputs that take the same branch are one test. |
| "I'll build a test helper first" | Write the test inline. Extract only after the third real duplicate, and only if it stays readable. |
| "The getter might change later" | Then the test that uses the getter will fail. A dedicated test for it catches nothing extra. |
| "I should verify the library works" | The library has its own tests. Test your configuration of it only where you did something non-obvious. |
| "The spec listed twelve scenarios" | Scenarios in a spec are requirements, not a test plan. Map them to outcomes, then test each outcome once. |
| "I'll just update the assertion so it passes again" | If behavior did not change, the test was pinned to implementation. Delete or re-aim it; patching keeps the churn source alive. |
