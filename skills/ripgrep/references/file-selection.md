# Choosing what gets searched

The difference between a 40-line answer and a 4,000-line one is almost always file
selection, not the pattern.

## What rg skips by default

1. Anything matched by `.gitignore`, `.ignore`, `.rgignore`, or `.git/info/exclude`
   (`.gitignore` only inside an actual git repo — see `--no-require-git`)
2. Hidden files and directories (dotfiles)
3. Binary files (search stops at the first NUL byte, with a note on stderr)
4. Symlinks (not followed)

Precedence, highest first: `-g/--glob` and `--iglob` beat everything → `.rgignore` →
`.ignore` → `.gitignore`. Later rules in a file beat earlier ones; deeper directories
beat shallower ones.

## Loosening filters — deliberately, and one step at a time

| Flag | Turns off |
|---|---|
| `--no-ignore-vcs` | `.gitignore` only (keeps `.ignore`/`.rgignore`) |
| `--no-ignore` / `-u` | all ignore files |
| `-.` / `--hidden` | the hidden-file skip |
| `-uu` | `--no-ignore --hidden` |
| `-uuu` | the above plus binary detection (`--binary`) — effectively "search everything" |
| `--no-require-git` | respect `.gitignore` even outside a git repo |

`-uu` is a **diagnostic**, not a default. Turning it on in a JS or Rust repo pulls in
`node_modules/` or `target/` and can produce tens of thousands of irrelevant hits from
vendored copies of the very code you're looking for — which then get reported as real
findings. If you must, scope it: `rg -uu 'pat' -g '!node_modules' -g '!target'`.

## Types

```bash
rg --type-list                  # every known type and its globs
rg --type-list | rg '^rust'     # what does `-t rust` actually cover?

rg -t rust    'pat'             # only Rust
rg -t js -t ts 'pat'            # union of types
rg -T test    'pat'             # exclude the `test` type
rg --type-add 'web:*.{js,ts,jsx,tsx,vue}' -t web 'pat'
```

Types are the cheapest and most legible filter. Prefer them over globs when one exists.

## Globs

```bash
rg -g '*.rs'         'pat'      # include
rg -g '!*_test.go'   'pat'      # exclude (leading !)
rg -g '**/api/**'    'pat'      # path segment
rg -g '*.{ts,tsx}'   'pat'      # brace alternation
rg --iglob '*.MD'    'pat'      # case-insensitive glob
```

Rules match `.gitignore` semantics and apply to the whole path. Multiple `-g` flags
combine; the last matching glob wins. `-g` **overrides ignore files** — this is how you
search a normally-ignored directory without going nuclear:

```bash
rg -n 'pat' -g 'dist/**' --no-ignore-vcs
```

## Scope by path, always

Positional paths are the strongest filter available and cost nothing:

```bash
rg -n 'pat' src/ tests/
rg -n 'pat' src/api/handlers.rs      # a single file is fine
rg -n 'pat' --max-depth 2 .
```

## Inspecting the file set before searching

`rg --files` prints exactly the files rg *would* search, with all filters applied. When a
search surprises you, this is the fastest way to find out why:

```bash
rg --files | wc -l                       # how big is the haystack?
rg --files -t rust | head
rg --files | rg 'config'                 # is the file I expect even in scope?
rg --files --no-ignore | rg 'secrets'    # was it ignored?
```

`--debug` explains which ignore rule excluded a given path.

## Piping into other tools

```bash
rg -l 'pat' | xargs rg -n 'other'        # AND across files
rg -l0 'pat' | xargs -0 <cmd>            # NUL-safe for paths with spaces
rg --files -t rust | xargs wc -l | tail -1
```

Note `-l0` is `-l --null`. Always use the NUL form when handing paths to another command.
