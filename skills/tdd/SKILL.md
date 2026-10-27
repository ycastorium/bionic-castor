---
name: tdd
description: Use when implementing a feature, bug fix, or behaviour change that has real logic in it; also when the user mentions TDD, red-green-refactor, or test-first.
---

# Test-Driven Development

## Philosophy

Tests verify behavior through public interfaces. Code can change entirely; tests should not.

A good test reads like a specification ("user can checkout with valid cart") and exercises a real code path through the public API. A bad test is coupled to implementation: it mocks internal collaborators, tests private methods, or breaks when you rename something even though behavior is unchanged.

Tests are code. Every one must be read, maintained, and fixed when it breaks. The goal is the smallest set of tests that catches the bugs you actually expect, not coverage. Fewer tests at a higher seam beat many tests at the function level.

See [tests.md](tests.md) for examples and [mocking.md](mocking.md) for mocking guidelines.

## When TDD Applies

TDD is for code with decisions in it: branching, calculations, validation, state transitions, anything at a trust boundary (auth, money, data writes), and every bug fix (one regression test that reproduces it).

Skip the loop entirely, and usually skip tests, for:

- Pass-through code: getters, constructors, delegation, thin wrappers, glue
- Config, constants, enum members, type shapes
- Framework or library behavior you did not configure in a non-obvious way
- Logging, metrics, debug output
- Private helpers; they are covered through the public behavior that uses them

Not every change needs the full RED/GREEN cycle. A small change inside already-tested behavior may need no new test at all, or only an extra assertion in an existing one. Use judgment; the default is "a test per decision", not "a test per change".

## Sizing

Before writing a test, name the bug it catches. If you cannot describe a realistic production mistake that would make it fail, do not write it.

- Tests track the number of decisions, not lines of code. A 200-line function with two branches needs about two tests
- One representative case per outcome plus the boundaries callers actually hit (empty, limit, the error the caller must handle). Ten inputs that take the same branch are one test
- A feature with three decisions usually needs three to five tests, not fifteen
- Stop when the next test would not change the implementation and would not catch a plausible bug

## Workflow

**Plan.** Agree on the public interface and list the behaviors worth testing, budgeted by the sizing rules above. If working from a task brief or approved design, the interface and acceptance criteria are already decided; do not re-ask. Use the project's domain vocabulary in test names and prefer [deep modules](deep-modules.md) with [testable interfaces](interface-design.md).

**Loop.** One test, then just enough code to pass it, then the next test. Do not write all tests up front; tests written in bulk test imagined behavior and end up asserting on shapes instead of outcomes. Each test should respond to what the previous cycle taught you.

```
RED:   write one test, run it, confirm it fails because the behavior is missing
GREEN: minimal code to pass, run it, output clean
repeat
```

Watch it fail at least once. A test that passes before the code exists is testing something already covered; delete or re-aim it. Derive expected values by hand; an expectation computed by the code under test proves nothing.

**Refactor.** Once GREEN, remove duplication and deepen modules ([refactoring.md](refactoring.md)). Run tests after each step. Never refactor while RED.

## Writing Tests That Last

- **Test at the seam that survives refactors**: the module or use-case boundary, not each function. If moving a function would require updating tests, they sit one level too low
- **Assert the minimum that proves the behavior**: the fields you care about, not deep equality on the whole object. Match error types, not message text. Do not assert order unless order is the behavior
- **Inline setup.** A few repeated arrange lines across tests is fine. No builders, base classes, or assertion DSLs until the same multi-line setup appears three or more times
- **One behavior per test, several assertions are fine.** Do not split one scenario into five tests
- **Hand-written fakes over mock expectations.** `called_with(...)` encodes the call graph; an in-memory fake at the boundary encodes only the contract
- **When a refactor breaks a test without changing behavior, do not patch the assertion.** Delete it or re-aim it higher. Delete tests along with the behavior they covered

## Quick Check Per Test

```
[ ] I can name the bug this catches
[ ] It uses the public interface and would survive an internal refactor
[ ] It is not an existing test with a different input
[ ] Setup is inline; expected values are literal or hand-checked
[ ] Watched it fail for the right reason
```

## Keep Both Failure Modes in View

| Under-testing | Over-testing |
|---------------|--------------|
| "Too simple" when it has a decision in it | A test for every input instead of every outcome |
| Bug fixed with no regression test | Tests for getters, config, and library behavior |
| Tests written after and never seen failing | Helpers and fixtures built before the third duplicate |
| Mocking because the interface is hard to use | Spec scenarios mapped one-to-one into tests |
| | Assertions patched so a refactor passes again |
