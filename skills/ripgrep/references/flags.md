# ripgrep flags, by task

Only the flags that change an answer or a cost. `rg --help` for the full list (it is
long); `rg -h` for the short one.

## Output form — pick one deliberately

| Flag | Effect |
|---|---|
| *(default)* | `path:text` when piped, `path:line:text` only on a TTY |
| `-n`, `--line-number` | force line numbers. **Always pass this.** |
| `-N`, `--no-line-number` | force them off (for `sort -u` pipelines) |
| `-H` / `-I` | force / suppress the filename prefix |
| `-l`, `--files-with-matches` | paths only |
| `--files-without-match` | inverse of `-l` — auditing missing headers, missing tests |
| `-c`, `--count` | matching **lines** per file |
| `--count-matches` | matching **occurrences** per file (differs when >1 per line) |
| `-q`, `--quiet` | no output, exit code only. Fastest existence check. |
| `-o`, `--only-matching` | print just the matched text, not the line |
| `-r REPL`, `--replace` | rewrite the **printed** match. Never touches files. `${1}` for groups. |
| `--vimgrep` | `path:line:col:text`, one line per match — good for machine parsing |
| `--json` | JSON Lines: `begin`/`match`/`end`/`summary`. Use for tooling, not for reading. |
| `--stats` | aggregate totals appended to the search |
| `--heading` / `--no-heading` | group by file (TTY default) vs `path:` prefix (pipe default) |
| `-0`, `--null` | NUL-terminate paths, for `xargs -0` |
| `--sort path` | deterministic order (single-threaded, slower). Use when diffing runs. |

## Context

| Flag | Effect |
|---|---|
| `-C N` | N lines before and after |
| `-B N` / `-A N` | before / after only |
| `--context-separator` | change the `--` divider |
| `--no-context-separator` | drop it |

Context multiplies output by ~`2N+1`. `-C2` is usually plenty; `-C10` is a decision you
should be able to justify.

## Bounding cost

| Flag | Effect |
|---|---|
| `-m N`, `--max-count` | stop after N matches **per file** — honest truncation |
| `--max-columns N` | skip absurdly long lines (minified JS, embedded blobs) |
| `--max-columns-preview` | show the first N columns instead of `[Omitted long line]` |
| `--max-depth N` | limit directory recursion |
| `--max-filesize 1M` | skip big files |
| `-j N`, `--threads` | rarely needed; `-j1` makes output order deterministic |

## Pattern input

| Flag | Effect |
|---|---|
| `-e PAT` | a pattern (repeatable — implicit OR). Also the escape hatch when a pattern starts with `-`. |
| `-f FILE` | read patterns from a file, one per line (`-f -` for stdin) |
| `-F`, `--fixed-strings` | treat patterns as literals |
| `-w`, `--word-regexp` | wrap in word boundaries |
| `-x`, `--line-regexp` | the pattern must match the whole line |
| `-v`, `--invert-match` | non-matching lines |
| `-i` / `-s` / `-S` | ignore case / force case-sensitive / smart-case |
| `-P`, `--pcre2` | PCRE2 engine: lookaround, backreferences |
| `-U`, `--multiline` | allow matches to cross line terminators |
| `--multiline-dotall` | make `.` cross `\n` (requires `-U`) |

`-e 'a' -e 'b'` is an OR. There is no AND — chain `rg -l a | xargs rg -n b`, or use
PCRE2 lookahead: `rg -P '(?=.*a)(?=.*b)'`.

## Exit codes

- `0` — matches found
- `1` — no matches (**not an error**; do not report this as a failure)
- `2` — an actual error (bad regex, unreadable path)

`rg -q pat && echo found || echo absent` is safe. `set -e` scripts must guard `rg`.

## Specialty

| Flag | Use |
|---|---|
| `-z`, `--search-zip` | search inside gz/bz2/xz/zstd without extracting |
| `--pre CMD` | preprocess each file through CMD (pdftotext, strings, ...). Pair with `--pre-glob`. |
| `-a`, `--text` | treat binary as text (rg otherwise stops at the first NUL) |
| `--binary` | search binaries but only report that a match exists |
| `--encoding utf-16` | force an encoding; `--encoding none` for raw bytes |
| `--follow` | follow symlinks (off by default; can loop) |
| `--type-add 'web:*.{js,ts,html}'` | define an ad-hoc type, then `-t web` |
| `--type-list` | what types exist and what they cover |

## Config file

`rg` reads flags from the file at `$RIPGREP_CONFIG_PATH`, one flag per line. Be aware it
exists: if a user's search behaves strangely, an inherited config may be adding
`--smart-case` or `--hidden`. `rg --no-config` (or `--debug`) to rule it out.
