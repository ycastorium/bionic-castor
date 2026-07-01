# Full Command Reference

Generated from `obsidian help`. Every option below is `key=value` unless marked as a flag (no value). Run `obsidian help <command>` for the live version, since community-plugin updates can add options.

## Vault & workspace

| Command | Options | Purpose |
|---|---|---|
| `vault` | `info=name\|path\|files\|folders\|size` | Show info about the active vault |
| `vaults` | `total`, `verbose` | List known vaults |
| `reload` | | Reload the vault |
| `restart` | | Restart the app |
| `workspace` | `ids` | Show workspace tree |
| `tabs` | `ids` | List open tabs |
| `tab:open` | `group=`, `file=`, `view=` | Open a new tab |

## Files & folders

| Command | Options | Purpose |
|---|---|---|
| `create` | `name=`, `path=`, `content=`, `template=`, `overwrite`, `open`, `newtab` | Create a new file |
| `read` | `file=`, `path=` | Read file contents |
| `append` | `file=`, `path=`, `content=`, `inline` | Append content to a file |
| `prepend` | `file=`, `path=`, `content=`, `inline` | Prepend content to a file |
| `open` | `file=`, `path=`, `newtab` | Open a file in the Obsidian UI |
| `move` | `file=`, `path=`, `to=` | Move or rename a file |
| `rename` | `file=`, `path=`, `name=` | Rename a file |
| `delete` | `file=`, `path=`, `permanent` | Delete a file (defaults to trash) |
| `file` | `file=`, `path=` | Show file info (size, timestamps) |
| `files` | `folder=`, `ext=`, `total` | List files in the vault |
| `folder` | `path=` (required), `info=files\|folders\|size` | Show folder info |
| `folders` | `folder=`, `total` | List folders in the vault |
| `wordcount` | `file=`, `path=`, `words`, `characters` | Count words/characters |
| `outline` | `file=`, `path=`, `format=tree\|md\|json`, `total` | Show headings for a file |

## Properties (frontmatter)

| Command | Options | Purpose |
|---|---|---|
| `properties` | `file=`, `path=`, `name=`, `total`, `sort=count`, `counts`, `format=yaml\|json\|tsv`, `active` | List properties in the vault or a file |
| `property:read` | `name=` (required), `file=`, `path=` | Read one property's value |
| `property:set` | `name=`, `value=` (required), `type=text\|list\|number\|checkbox\|date\|datetime`, `file=`, `path=` | Set a property |
| `property:remove` | `name=` (required), `file=`, `path=` | Remove a property |
| `aliases` | `file=`, `path=`, `total`, `verbose`, `active` | List aliases (a specific property) |

## Tags

| Command | Options | Purpose |
|---|---|---|
| `tags` | `file=`, `path=`, `total`, `counts`, `sort=count`, `format=json\|tsv\|csv`, `active` | List tags in the vault or a file |
| `tag` | `name=` (required), `total`, `verbose` | Show occurrences of one tag |

## Tasks

| Command | Options | Purpose |
|---|---|---|
| `tasks` | `file=`, `path=`, `total`, `done`, `todo`, `status="<char>"`, `verbose`, `format=json\|tsv\|csv`, `active`, `daily` | List tasks (checkbox lines) |
| `task` | `ref=<path:line>`, `file=`, `path=`, `line=`, `toggle`, `done`, `todo`, `daily`, `status="<char>"` | Show or update a single task |

`ref=path:line` is the fastest way to target one task once `tasks` has given a line number.

## Daily notes

| Command | Options | Purpose |
|---|---|---|
| `daily` | `paneType=tab\|split\|window` | Open today's daily note |
| `daily:path` | | Print today's daily note path |
| `daily:read` | | Read today's daily note |
| `daily:append` | `content=` (required), `inline`, `open`, `paneType=` | Append to today's daily note |
| `daily:prepend` | `content=` (required), `inline`, `open`, `paneType=` | Prepend to today's daily note |

## Links & graph

| Command | Options | Purpose |
|---|---|---|
| `links` | `file=`, `path=`, `total` | List outgoing links from a file |
| `backlinks` | `file=`, `path=`, `counts`, `total`, `format=json\|tsv\|csv` | List backlinks to a file |
| `unresolved` | `total`, `counts`, `verbose`, `format=json\|tsv\|csv` | List links with no target note |
| `orphans` | `total`, `all` | List files with no incoming links |
| `deadends` | `total`, `all` | List files with no outgoing links |

## Search

| Command | Options | Purpose |
|---|---|---|
| `search` | `query=` (required), `path=`, `limit=`, `total`, `case`, `format=text\|json` | Search vault text, file-level results |
| `search:context` | `query=` (required), `path=`, `limit=`, `case`, `format=text\|json` | Search with matching line context |
| `search:open` | `query=` | Open the search view with a query |

## Bookmarks, history, recents

| Command | Options | Purpose |
|---|---|---|
| `bookmark` | `file=`, `subpath=`, `folder=`, `search=`, `url=`, `title=` | Add a bookmark |
| `bookmarks` | `total`, `verbose`, `format=json\|tsv\|csv` | List bookmarks |
| `recents` | `total` | List recently opened files |
| `history` | `file=`, `path=` | List version history for a file |
| `history:list` | | List files with history |
| `history:read` | `file=`, `path=`, `version=` | Read a history version |
| `history:restore` | `file=`, `path=`, `version=` (required) | Restore a history version |
| `history:open` | `file=`, `path=` | Open file recovery UI |
| `diff` | `file=`, `path=`, `from=`, `to=`, `filter=local\|sync` | Diff local/sync versions |

## Sync

| Command | Options | Purpose |
|---|---|---|
| `sync` | `on`, `off` | Pause/resume sync |
| `sync:status` | | Show sync status |
| `sync:history` | `file=`, `path=`, `total` | List sync version history |
| `sync:read` | `file=`, `path=`, `version=` (required) | Read a sync version |
| `sync:restore` | `file=`, `path=`, `version=` (required) | Restore a sync version |
| `sync:deleted` | `total` | List deleted files known to sync |
| `sync:open` | `file=`, `path=` | Open sync history UI |

## Templates & bases

| Command | Options | Purpose |
|---|---|---|
| `templates` | `total` | List templates |
| `template:read` | `name=` (required), `resolve`, `title=` | Read a template, optionally resolving variables |
| `template:insert` | `name=` (required) | Insert a template into the active file |
| `bases` | | List base files (Obsidian Bases feature) in the vault |
| `base:views` | | List views in the current base file |
| `base:query` | `file=`, `path=`, `view=`, `format=json\|csv\|tsv\|md\|paths` | Query a base and return results |
| `base:create` | `file=`, `path=`, `view=`, `name=`, `content=`, `open`, `newtab` | Create a new item in a base |

## Commands, hotkeys, plugins, themes, snippets

| Command | Options | Purpose |
|---|---|---|
| `commands` | `filter=<prefix>` | List available Obsidian commands (by ID) |
| `command` | `id=` (required) | Execute an Obsidian command by ID |
| `hotkey` | `id=` (required), `verbose` | Get the hotkey bound to a command |
| `hotkeys` | `total`, `verbose`, `format=json\|tsv\|csv`, `all` | List hotkeys |
| `plugin` | `id=` (required) | Get plugin info |
| `plugins` | `filter=core\|community`, `versions`, `format=json\|tsv\|csv` | List installed plugins |
| `plugins:enabled` | `filter=`, `versions`, `format=` | List enabled plugins |
| `plugin:enable` / `plugin:disable` | `id=` (required), `filter=` | Toggle a plugin |
| `plugin:install` / `plugin:uninstall` | `id=` (required), `enable` | Install/remove a community plugin |
| `plugin:reload` | `id=` (required) | Reload a plugin (development) |
| `plugins:restrict` | `on`, `off` | Toggle restricted mode |
| `themes` | `versions` | List installed themes |
| `theme` | `name=` | Show active theme or theme details |
| `theme:set` | `name=` (required, empty for default) | Set active theme |
| `theme:install` / `theme:uninstall` | `name=` (required), `enable` | Install/remove a theme |
| `snippets` / `snippets:enabled` | | List CSS snippets |
| `snippet:enable` / `snippet:disable` | `name=` (required) | Toggle a CSS snippet |

## Developer commands

Only relevant when debugging the Obsidian app itself, not for note-taking workflows:

`dev:cdp`, `dev:console`, `dev:css`, `dev:debug`, `dev:dom`, `dev:errors`, `dev:mobile`, `dev:screenshot`, `devtools`, `eval`.

`eval code=<javascript>` runs arbitrary JS inside the Obsidian renderer — powerful but treat it like running a script with full access to the user's vault and Electron app; confirm intent before using it destructively.
