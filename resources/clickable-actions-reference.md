# Clickable Actions Reference

This guide covers the use of clickable actions in workshop instruction pages. Clickable actions provide interactive elements that users can click to execute commands, copy text, edit files, and more.

For detailed syntax and examples of each action type, see the category-specific reference files linked below.

## How Clickable Actions Work

Clickable actions are implemented as fenced Markdown code blocks with special language types. The language type determines the action behavior, and the block body contains YAML configuration.

**Basic syntax:**

````markdown
```<action-type>
<yaml-body>
```
````

**Example:**

````markdown
```terminal:execute
command: kubectl get pods
```
````

## YAML Syntax Safety for Clickable Actions

Because the body of every clickable action is parsed as YAML, you must ensure the content is valid YAML. Two common categories of mistakes cause broken workshop instructions: YAML-special characters in values, and invalid shell quoting in commands.

### YAML-special characters in command values

YAML treats certain characters as syntax when they appear in unquoted scalar values. The most common problem is a **colon followed by a space** (`: `), which YAML interprets as a mapping key-value separator. Other problematic characters include `#` (comment), `{` `}` (flow mapping), `[` `]` (flow sequence), `>` `|` (block scalars), `&` `*` (anchors/aliases), `!` (tags), `%` (directives), and backticks.

When a `command:` value or any other string property contains these characters, the YAML parser will either error or silently misparse the content. The fix is to use YAML block scalar syntax (`|-` for strip-trailing-newline, or `|` for keep-trailing-newline) instead of putting the value on the same line as the key.

**WRONG — colon in command breaks YAML parsing:**

````markdown
```terminal:execute
command: docker run -p 8080:80 nginx:latest
```
````

**CORRECT — block scalar avoids the problem:**

````markdown
```terminal:execute
command: |-
  docker run -p 8080:80 nginx:latest
```
````

**WRONG — hash character starts a YAML comment:**

````markdown
```terminal:execute
command: echo hello # this is not a yaml comment
```
````

**CORRECT:**

````markdown
```terminal:execute
command: |-
  echo hello # this is a shell comment, safe inside block scalar
```
````

**Rule of thumb:** If a command contains any of `: # { } [ ] > | & * ! % @` or backticks, always use block scalar syntax (`command: |-`). When in doubt, use block scalar syntax — it is always safe and never hurts.

For multi-line commands (heredocs, pipelines, etc.), block scalar syntax is required:

````markdown
```terminal:execute
command: |-
  cat << 'EOF' > config.yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: my-service
  EOF
```
````

### Shell quoting in commands

Commands executed via `terminal:execute` run in a shell. The command string must be valid shell syntax. Common mistakes:

**WRONG — unquoted string with spaces:**

````markdown
```terminal:execute
command: |-
  echo Hello World > /tmp/my file.txt
```
````

**CORRECT — proper shell quoting:**

````markdown
```terminal:execute
command: |-
  echo "Hello World" > "/tmp/my file.txt"
```
````

**WRONG — unquoted variable that may contain spaces:**

````markdown
```terminal:execute
command: |-
  cat $HOME/my documents/file.txt
```
````

**CORRECT:**

````markdown
```terminal:execute
command: |-
  cat "$HOME/my documents/file.txt"
```
````

Always test that commands work correctly in a terminal before embedding them in workshop instructions.

## All Clickable Action Types

### Terminal Actions

For details see [clickable-actions/terminal-actions.md](clickable-actions/terminal-actions.md).

| Action Type | Purpose |
|-------------|---------|
| `terminal:execute` | Execute a command in a terminal session |
| `terminal:execute-all` | Execute a command in all terminal sessions |
| `terminal:input` | Send text to a terminal (for interactive prompts) |
| `terminal:interrupt` | Send Ctrl+C to a terminal session |
| `terminal:interrupt-all` | Send Ctrl+C to all terminal sessions |
| `terminal:clear` | Clear a terminal session buffer |
| `terminal:clear-all` | Clear all terminal session buffers |
| `terminal:select` | Select a terminal session and give it keyboard focus |

### Editor File Actions

For details see [clickable-actions/editor-file-actions.md](clickable-actions/editor-file-actions.md).

| Action Type | Purpose |
|-------------|---------|
| `editor:open-file` | Open a file in the editor |
| `editor:create-file` | Create a new file or overwrite an existing file |
| `editor:create-directory` | Create a directory |
| `editor:append-lines-to-file` | Append text to end of file (creates file if missing) |
| `editor:insert-lines-before-line` | Insert text before a line number |
| `editor:insert-lines-after-line` | Insert text after a line number |
| `editor:append-lines-after-match` | Insert text after a line matching a string |
| `editor:select-matching-text` | Select text by exact match or regex |
| `editor:select-lines-in-range` | Select lines by line number range |
| `editor:replace-text-selection` | Replace currently selected text |
| `editor:replace-matching-text` | Find and replace in a single step |
| `editor:replace-lines-in-range` | Replace a range of lines |
| `editor:delete-lines-in-range` | Delete a range of lines |
| `editor:delete-matching-lines` | Delete lines matching a string or regex |
| `editor:copy-file` | Copy a file to a new location |
| `editor:rename-file` | Rename or move a file |
| `editor:close-file` | Close a file tab in the editor |
| `editor:delete-file` | Delete a file from the filesystem |

### Editor YAML Actions

For details see [clickable-actions/editor-yaml-actions.md](clickable-actions/editor-yaml-actions.md).

These provide structured YAML manipulation that preserves comments and handles all YAML styles. They replace the deprecated `editor:insert-value-into-yaml`.

| Action Type | Purpose |
|-------------|---------|
| `editor:set-yaml-value` | Set or update a value at a YAML path |
| `editor:add-yaml-item` | Append an item to a YAML sequence |
| `editor:insert-yaml-item` | Insert an item at a position in a YAML sequence |
| `editor:replace-yaml-item` | Replace a sequence item by index or attribute match |
| `editor:delete-yaml-value` | Delete a key or sequence item |
| `editor:merge-yaml-values` | Merge key-value pairs into a YAML mapping |
| `editor:select-yaml-path` | Select (highlight) a YAML node in the editor |

### Editor Terminal Actions

For details see [clickable-actions/editor-terminal-actions.md](clickable-actions/editor-terminal-actions.md).

These manage terminals within the VS Code editor, separate from dashboard terminals. Editor terminals use named string sessions (e.g., `"educates"`, `"build"`) rather than the integer session numbers used by dashboard `terminal:*` actions.

| Action Type | Purpose |
|-------------|---------|
| `editor:open-terminal` | Open or create a terminal in the editor |
| `editor:send-to-terminal` | Send text/command to an editor terminal |
| `editor:interrupt-terminal` | Send Ctrl+C to an editor terminal |
| `editor:clear-terminal` | Clear an editor terminal buffer |
| `editor:close-terminal` | Close and dispose of an editor terminal |
| `editor:execute-command` | Execute a registered VS Code command |

### Dashboard Actions

For details see [clickable-actions/dashboard-actions.md](clickable-actions/dashboard-actions.md).

| Action Type | Purpose |
|-------------|---------|
| `dashboard:open-url` | Open a URL in a new browser tab |
| `dashboard:open-dashboard` | Show/focus a dashboard tab |
| `dashboard:create-dashboard` | Create a new dashboard tab (URL or terminal) |
| `dashboard:reload-dashboard` | Reload a dashboard tab (or create if missing) |
| `dashboard:delete-dashboard` | Delete a custom dashboard tab |

### Section Actions

For details see [clickable-actions/section-actions.md](clickable-actions/section-actions.md).

| Action Type | Purpose |
|-------------|---------|
| `section:begin` | Start a collapsible section |
| `section:end` | End a collapsible section |
| `section:heading` | Non-collapsible styled heading |

### File Transfer Actions

For details see [clickable-actions/file-transfer-actions.md](clickable-actions/file-transfer-actions.md).

Require file download/upload to be enabled in the workshop configuration.

| Action Type | Purpose |
|-------------|---------|
| `files:download-file` | Download a file to the user's local computer |
| `files:copy-file` | Copy file contents to the user's clipboard |
| `files:upload-file` | Upload a single file with a predetermined name |
| `files:upload-files` | Upload one or more files keeping original names |

### Examiner Actions

For details see [clickable-actions/examiner-actions.md](clickable-actions/examiner-actions.md).

Require the test examiner to be enabled in the workshop configuration.

| Action Type | Purpose |
|-------------|---------|
| `examiner:execute-test` | Run a verification test script |

### Copy Actions

| Action Type | Purpose |
|-------------|---------|
| `workshop:copy` | Copy text to the system clipboard |
| `workshop:copy-and-edit` | Copy text to clipboard, visually flagged as needing editing |

Both actions accept a single property:

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `text` | string | (required) | The text to copy to the clipboard |

Use `workshop:copy` when the text can be pasted verbatim. Use `workshop:copy-and-edit` when the text contains a placeholder the learner must customize before pasting (e.g., a username or personal token). The `copy-and-edit` variant renders with a visual indicator reminding the learner to edit the value.

**Example — verbatim copy:**

````markdown
```workshop:copy
text: export MY_VAR=some-fixed-value
```
````

**Example — copy with editing required:**

````markdown
```workshop:copy-and-edit
text: export MY_TOKEN=replace-with-your-token
```
````

## Common Properties

These optional properties can be added to any clickable action that accepts YAML. Do not set `prefix`, `title`, or `description` unless you need to override the default display — the system generates sensible defaults.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `prefix` | string | (auto) | Prefix for the banner (e.g., "Run", "Section") |
| `title` | string | (auto) | Title text in the banner |
| `description` | string | — | Additional text below the banner (always pre-formatted) |
| `autostart` | boolean | `false` | Trigger the action automatically on page load (or section expand) |
| `cascade` | boolean | `false` | Trigger the next clickable action when this one succeeds |
| `hidden` | boolean | `false` | Hide the action from view (useful with `autostart`) |
| `cooldown` | number | `3` | Seconds before action can be re-clicked. `.INF` or `-1` to prevent re-clicking |
| `event` | string/object | — | Analytics event name or object sent to the registry webhook |

## Deprecated Clickable Actions

Do NOT use these. Use the recommended alternatives:

| Deprecated Action | Use Instead |
|-------------------|-------------|
| `execute` | `terminal:execute` |
| `execute-1`, `execute-2`, `execute-3` | `terminal:execute` with `session` property |
| `execute-all` | `terminal:execute-all` |
| `copy` | `workshop:copy` |
| `copy-and-edit` | `workshop:copy-and-edit` |
| `editor:insert-value-into-yaml` | `editor:set-yaml-value`, `editor:add-yaml-item`, `editor:insert-yaml-item`, `editor:replace-yaml-item`, `editor:delete-yaml-value`, `editor:merge-yaml-values` |
