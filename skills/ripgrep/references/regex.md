# Patterns: escaping, engines, multiline

## Two layers of escaping

Every pattern passes through the **shell** and then the **regex engine**. Most "rg
returned nothing" bugs are one of these two, not rg.

**Shell:** single-quote everything. `'...'` is literal in POSIX shells (only `'` itself
needs care). Double quotes expand `$var`, `` `cmd` ``, `\`, and history `!`.

```bash
rg -n 'price = \$(\d+)'      # correct
rg -n "price = \$(\d+)"      # works, but you had to escape $ twice — don't
rg -n "$1"                   # BUG: the shell substituted a positional arg
```

Pattern starts with `-`? Use `-e` or `--`:

```bash
rg -e '-Wall' .
rg -- '-Wall' .
```

**Regex:** these are all metacharacters — `. ^ $ * + ? ( ) [ ] { } | \`. If your target
contains any of them and you don't mean them as regex, use `-F`.

```bash
rg -nF 'app.config["db.url"]'          # literal
rg -n  'app\.config\["db\.url"\]'      # same thing, worse
```

## Which engine

ripgrep defaults to the **Rust regex crate**: linear time, no backtracking, no
lookaround, no backreferences. `-P/--pcre2` (or `--engine=pcre2`) switches to PCRE2;
`--engine=auto` falls back to PCRE2 only when the default engine rejects the pattern.

Use `-P` when — and only when — you need:

- lookahead `(?=...)` / negative lookahead `(?!...)`
- lookbehind `(?<=...)` / `(?<!...)`
- backreferences `\1`
- atomic groups, recursion

```bash
# lines mentioning `token` but not in a comment
rg -nP '^(?!\s*(//|#)).*token'

# an AND across a line
rg -nP '(?=.*async)(?=.*retry)'
```

PCRE2 is slower and can backtrack pathologically on adversarial patterns. Prefer two
piped `rg` calls over a clever lookaround when the intent is "files with A and B":

```bash
rg -l 'async' | xargs rg -n 'retry'
```

## Multiline

rg is line-oriented by default; `\n` cannot appear in a match.

- `-U`, `--multiline` — lift the restriction. `\n` and `\p{any}` now work.
- `--multiline-dotall` — additionally make `.` match `\n`.

```bash
# a function signature spanning lines
rg -nU 'fn build\([^)]*\n[^)]*\)' src/

# a whole block, lazily
rg -nU --multiline-dotall 'impl Config \{.*?\n\}' src/
```

Caveats: multiline mode buffers whole files (more memory), and `-U` cannot be combined
with `--line-regexp` or line-based short-circuiting. Match *counts* are per match, not
per line. Greedy `.*` with `--multiline-dotall` will happily swallow the entire file —
use `.*?`.

## Useful syntax

| Want | Pattern |
|---|---|
| Word boundary | `\bfoo\b`, or just `-w` |
| Start/end of line | `^` / `$` (per-line, even under `-U`) |
| Non-greedy | `.*?`, `.+?` |
| Alternation | `(foo\|bar)` — quote it, `\|` is a shell pipe unquoted |
| Digits / word / space | `\d` `\w` `\s` (Unicode-aware by default) |
| Unicode category | `\p{Lu}`, `\p{Greek}` |
| Case-insensitive fragment | `(?i)foo` inline, or `-i` globally |
| Literal fragment inside a regex | `\Qfoo.bar\E` — **PCRE2 only**, needs `-P`. The default engine rejects it. |
| Repeat | `\w{3,8}` |

`--no-unicode` disables Unicode classes if you need byte semantics (rare, and it makes
`\w`/`\b` ASCII-only).

## CRLF

On files with Windows line endings, `$` will not match before `\r` unless you pass
`--crlf`. Symptom: `rg -n 'foo$'` finds nothing in a checkout with CRLF files.

## Replacement strings (`-r`)

Only relevant for reformatting output — **rg never writes files**.

- `$0` = whole match, `$1`, `$2` = groups by opening-paren order, `$name` = named group.
- **Always brace when followed by word characters**: `$1a` parses as the group named
  `1a` and expands to *empty*. Write `${1}a`.
- A reference to a group that didn't participate expands to the empty string, silently.
- A literal `$` in the replacement is `$$`.

```bash
echo 'foo123' | rg -o -r '${1}-x' '([a-z]+)'   # foo-x
echo 'foo123' | rg -o -r '$1-x'   '([a-z]+)'   # foo-x  (- is not a word char, fine)
echo 'foo123' | rg -o -r '$1x'    '([a-z]+)'   # (empty!) — group "1x" doesn't exist
```
