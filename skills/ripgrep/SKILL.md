---
name: ripgrep
description: >
  Use whenever searching a codebase or any file tree with `rg` / ripgrep — finding
  symbols, call sites, string literals, config values, TODOs, or auditing patterns
  across many files. Triggers on "ripgrep", "rg", "grep for", "search the codebase
  for", "find all usages of", "which files contain", "why did my rg return nothing",
  "rg found too much", or any shell command starting with `rg`. Covers the flags that
  matter, correct quoting and regex escaping, file/type selection, output shaping for
  limited context windows, and the misuses that silently produce wrong answers —
  especially `-r/--replace`, which never edits files.
---

# Using ripgrep (`rg`) correctly

`rg` is the default search tool, and it is *very* easy to use in a way that returns a
confidently wrong answer: zero hits because a metacharacter was unescaped, a truncated
picture because `.gitignore` hid the file, or an unreadable wall of text that eats the
context window. This skill is about getting the *right* answer cheaply.

## Golden rules

1. **`rg` never modifies files.** Not with any flag. `-r/--replace` only changes what is
   *printed*. To edit files, use the `edit` tool. See "The `-r` trap" below.
2. **Always pass `-n`.** When `rg` output is piped or captured (which is always, for an
   agent), line numbers are **off by default**. Without `-n` you get `path:text` and no
   way to cite a location. This is the single most common silent defect.
3. **Searching for a literal? Use `-F`.** Any pattern containing `. ( ) [ ] { } * + ? | ^ $ \`
   is a regex unless you say otherwise. `rg -F 'foo(bar)'` is right; `rg 'foo(bar)'`
   searches for `foobar`.
4. **Single-quote the pattern.** `'...'` in POSIX shells. Double quotes let the shell eat
   `$1`, `$foo`, backticks, and `!`. Never write `rg "$something"` unless you mean
   shell interpolation.
5. **Shape output before you run it.** Decide up front whether you want *paths* (`-l`),
   *counts* (`-c`), *lines* (default), or *context* (`-C2`). Dumping unbounded matches
   into your context is a cost, not thoroughness.
6. **Zero results is a claim that needs verification.** Before concluding "X doesn't
   exist", re-run once with `-i` and once with `-uu` (see "Debugging zero hits").
7. **`rg` finds text, not structure.** For "every call to this function with 3 args" or
   "every struct implementing this trait", `rg` gives you candidates, not answers. Read
   the candidates before asserting.

## The three misuses to stop making

### 1. The `-r` trap — using replace when you wanted to search

`-r/--replace` exists to reformat *displayed* output. It does not touch disk. The
ripgrep man page says it outright: *"Neither this flag nor any other ripgrep flag will
modify your files."*

Two failure modes:

- **Believing it edited something.** It did not. Nothing was written. If you need to
  rewrite files, use the `edit` tool with exact `oldText`, or `sd`/`sed -i` if the user
  explicitly wants a bulk shell rewrite.
- **Reaching for `-r` (or `-o`) to plain-search.** If you just want to see matches, the
  default output is what you want. `-r` *destroys* the surrounding line, which is
  usually the information you actually needed.

```bash
# WRONG — you wanted to see where the function is used; now you see only the name back
rg -o -r '$1' 'fn (\w+)' src/

# RIGHT
rg -n 'fn \w+' src/
```

Legitimate uses of `-r` are narrow: extracting one field from structured lines for a
pipeline, and normalizing output for a diff. Both should be paired with `-o` and piped
into something.

```bash
# Legit: collect every crate version declared in Cargo.toml files
rg -No -r '$1' '^version = "([^"]+)"' --glob 'Cargo.toml' | sort -u
```

Capture-group gotcha in replacements: `$1a` means *the group named `1a`*, not group 1
followed by `a` — and an unknown group expands to the empty string, silently. Always
brace it: `${1}a`.

### 2. Regex and quoting mistakes

| Symptom | Cause | Fix |
|---|---|---|
| Zero hits on an obviously present string | Unescaped `.` `(` `[` `*` `?` `+` `$` | `-F`, or escape it |
| Zero hits searching a path | `/` is fine, but `.` and `-` are not | `rg -F 'src/foo-bar.rs'` |
| `$1` vanished | Double quotes; the shell expanded it | Single-quote the pattern |
| `error: unrecognized flag` | Pattern begins with `-` | `rg -e '-Wall'` or `rg -- '-Wall'` |
| Lookahead/lookbehind errors | Rust regex has no backtracking | `-P` (PCRE2) |
| Pattern can't match across lines | Line-oriented by default | `-U` (+ `--multiline-dotall` if `.` must cross `\n`) |
| Matches `set` inside `offset` | No word boundary | `-w`, or `\b` |

Reach for `-P` only when you actually need lookaround or backreferences; it is slower
and gives up some of rg's optimizations. Full detail: `references/regex.md`.

### 3. Bad file selection and output noise

- **Filter by type, not by hope.** `rg -t rust 'foo'` beats `rg 'foo' --glob '*.rs'`
  beats grepping everything and eyeballing. `rg --type-list` shows what's available;
  `-T <type>` excludes.
- **`--glob` for what types can't express**: `-g '!**/tests/**'`, `-g '*.{ts,tsx}'`.
  Negations start with `!`. Globs are matched against the whole path.
- **Don't reach for `--no-ignore` / `-uu` reflexively.** rg respects `.gitignore` for a
  reason: it keeps `node_modules`, `target/`, `dist/`, and lockfiles out of your context.
  Use `-uu` as a *diagnostic* when a search unexpectedly returns nothing, then go back.
- **Cap the blast radius.** `-m 5` (max matches per file), `--max-columns 200
  --max-columns-preview` (survive minified files), `-l` when you only need the file set.
- **Scope the path.** `rg -n 'foo' src/api/` is almost always better than `rg -n 'foo'`
  from the repo root.

```bash
# WRONG — searches vendored deps, prints minified megabytes, no line numbers
rg --no-ignore 'handleRequest'

# RIGHT — narrow, cited, bounded
rg -n -t ts 'handleRequest' src/
```

## Recipes

```bash
# Where is this defined? (symbol, with 2 lines of context)
rg -n -C2 -t rust 'fn parse_header' src/

# Which files mention it at all? (cheap survey before reading anything)
rg -l 'parse_header'

# How much of a problem is this? (counts per file, then decide)
rg -c 'unwrap\(\)' src/ | sort -t: -k2 -rn | head

# Exact literal, including punctuation
rg -nF 'config["db.url"]'

# Whole word only — avoids `id` matching `uuid`, `width`, ...
rg -nw 'id' src/

# Several alternatives at once
rg -n -e 'TODO' -e 'FIXME' -e 'HACK' src/

# Multi-line: a struct definition body
rg -nU --multiline-dotall 'struct Config \{.*?\}' src/

# Lookbehind (needs PCRE2): assignments to `port` that aren't comments
rg -nP '(?<!//\s)port\s*=' src/

# Only the matched text, deduplicated — e.g. every env var referenced
rg -No '\bENV_[A-Z_]+' | sort -u

# Files that do NOT contain a required header
rg --files-without-match 'SPDX-License-Identifier' -t rust

# Just list what rg would search (verify your filters before searching)
rg --files -t rust src/ | head

# Sanity/aggregate check on a big audit
rg --stats -t py 'except:' .
```

## Debugging zero hits

Run this ladder, stopping at the first one that produces output:

1. `rg -nF '<literal>'` — was it a regex escaping problem?
2. `rg -ni '<pattern>'` — was it case?
3. `rg -n '<pattern>' --no-ignore` — was it `.gitignore`?
4. `rg -n '<pattern>' -uu` — was it a hidden file or ignore rule?
5. `rg --files | rg '<expected-file>'` — is the file even in scope? (wrong cwd is common)
6. `rg -nU '<pattern>'` — does the match span lines?

Then report *which* rung fixed it. "Not found" without this ladder is an unverified claim.

## Output shaping for a limited context window

| You need | Use | Cost |
|---|---|---|
| Does it exist at all? | `rg -q 'pat' && echo yes` | ~0 |
| Which files? | `rg -l 'pat'` | one line per file |
| How many? | `rg -c 'pat'` / `--count-matches` | one line per file |
| Matches with locations | `rg -n 'pat'` | one line per match |
| Enough to understand | `rg -n -C2 'pat'` | 5 lines per match |
| Everything, bounded | `rg -n 'pat' -m 3 --max-columns 200` | bounded |

If a search might return hundreds of hits, run `rg -c` first, then narrow. Piping to
`head` truncates *arbitrarily* and hides that you truncated — prefer `-m`, or say so.

## References

- `references/flags.md` — task-oriented flag reference, including the ones worth knowing
  (`--json`, `--sort`, `-z`, `--pre`, `--vimgrep`, `--null`, `-f`).
- `references/regex.md` — Rust-regex vs PCRE2, escaping, multiline, Unicode classes.
- `references/file-selection.md` — types, globs, ignore-file precedence, `-u` levels.
