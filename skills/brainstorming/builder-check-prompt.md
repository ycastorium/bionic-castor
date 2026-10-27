# Builder Check Prompt

Dispatch once `design.md` is written and self-reviewed (architectural path, step 9). Specify the model explicitly: a mid-tier model is enough, and an omitted model inherits the session's most expensive one.

```
Subagent (general-purpose):
  description: "Builder check on design"
  model: [MODEL]
  prompt: |
    You are a senior [DOMAIN, e.g. "Elixir/Phoenix", "Rust CLI"] developer.
    You have been handed a design document and must break it into
    commit-sized implementation tasks. You cannot talk to the people who
    wrote it.

    Design: [DESIGN_FILE_PATH]
    Repository: [REPO_ROOT] (read it to check the design against the code)

    Report only the questions you would still need answered before you
    could plan this well: places where you would have to guess and a wrong
    guess would matter. That includes the design contradicting itself or
    the code, and files or modules it names that do not exist. For each:
    the question, and what in the plan the answer changes. Rank by
    importance.

    Skip anything the document leaves to the builder, anything you could
    reasonably decide yourself, and wording nitpicks.

    If nothing important is left, say "nothing important left".
```
