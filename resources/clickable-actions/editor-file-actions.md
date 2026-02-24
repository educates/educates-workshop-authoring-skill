# Editor File Clickable Actions

Actions for opening, creating, editing, and managing files through the embedded editor. These require the editor to be enabled in the workshop session.

## editor:open-file

Opens a file in the editor. Optionally positions the cursor on a specific line.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path. Use `~/` or `$HOME/` for home directory |
| `line` | integer | — | Line number to position cursor on (1-indexed) |

Note: When the embedded editor is enabled in a workshop, prefer using this action to display file contents to users rather than using terminal commands like `cat`.

**Example:**

````markdown
```editor:open-file
file: ~/exercises/deployment.yaml
line: 8
```
````

## editor:create-file

Creates a new file with the specified content, or overwrites an existing file entirely. The containing directory must already exist.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `text` | string | (required) | Content for the file |

**Example:**

````markdown
```editor:create-file
file: ~/exercises/config.yaml
text: |
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: example
  data:
    key: value
```
````

## editor:create-directory

Creates a directory. Use this before `editor:create-file` or `editor:append-lines-to-file` if the target directory does not exist.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `directory` | string | (required) | Directory path to create |

**Example:**

````markdown
```editor:create-directory
directory: ~/exercises/configs
```
````

## editor:append-lines-to-file

Appends text to the end of a file. If the file does not exist, it will be created.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `text` | string | (required) | Text to append |

Note: If it is known the file does not already exist, use `editor:create-file` instead for clearer semantic intent.

**Example:**

````markdown
```editor:append-lines-to-file
file: ~/exercises/notes.txt
text: |
  Additional notes added at the end.
```
````

## editor:insert-lines-before-line

Inserts text before a specified line number.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `line` | integer | (required) | Line number to insert before (1-indexed) |
| `text` | string | (required) | Text to insert |

**Example:**

````markdown
```editor:insert-lines-before-line
file: ~/exercises/app.py
line: 5
text: |
  import logging
  logger = logging.getLogger(__name__)
```
````

## editor:insert-lines-after-line

Inserts text after a specified line number.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `line` | integer | (required) | Line number to insert after (1-indexed) |
| `text` | string | (required) | Text to insert |

**Example:**

````markdown
```editor:insert-lines-after-line
file: ~/exercises/app.py
line: 3
text: |
  import os
```
````

## editor:append-lines-after-match

Inserts text after the first line containing a matching string.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `match` | string | (required) | Text to search for in lines |
| `text` | string | (required) | Text to insert after the matching line |

**Example:**

````markdown
```editor:append-lines-after-match
file: ~/exercises/requirements.txt
match: flask
text: |
  flask-cors==4.0.0
```
````

## editor:select-matching-text

Selects (highlights) text in a file based on exact match or regular expression. Useful as a visual aid or as a precursor to `editor:replace-text-selection`.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `text` | string | (required) | Text or regex pattern to match |
| `isRegex` | boolean | `false` | Treat `text` as a regular expression |
| `group` | integer | — | Regex subgroup to select (when using regex) |
| `before` | integer | — | Extra lines to highlight before match. `0` = full line, `-1` = all lines before |
| `after` | integer | — | Extra lines to highlight after match. `0` = full line, `-1` = all lines after |
| `start` | integer | — | Start line for search range (1-indexed). Negative = offset from end |
| `stop` | integer | — | End line for search range (exclusive). Negative = offset from end |

**Example — exact match:**

````markdown
```editor:select-matching-text
file: ~/exercises/deployment.yaml
text: "image: nginx:1.19"
```
````

**Example — regex with subgroup:**

````markdown
```editor:select-matching-text
file: ~/exercises/deployment.yaml
text: "image: (.*)"
isRegex: true
group: 1
```
````

### Multi-line matching

The `text` property can span multiple lines, allowing you to select an entire block of content in the editor.

**Example — select a multi-line block:**

````markdown
```editor:select-matching-text
file: ~/exercises/deployment.yaml
text: |
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
```
````

When the text to match starts with leading spaces (because it is indented within the file), use the YAML block scalar **indent indicator** to preserve those spaces. The digit after `|` specifies how many spaces form the indentation level:

**Example — select indented content using indent indicator:**

````markdown
```editor:select-matching-text
file: ~/exercises/deployment.yaml
text: |2
      containers:
      - name: app
        image: myapp:v1
```
````

Without the indent indicator (`|2`), the YAML parser strips the leading spaces and the match fails against the indented content in the file. See the [indent indicator note under `editor:replace-matching-text`](#multi-line-matching-and-replacement) for a fuller explanation.

## editor:select-lines-in-range

Selects a range of lines by line number. The selected text can then be replaced using `editor:replace-text-selection`.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `start` | integer | (required) | First line to select (1-indexed, inclusive) |
| `stop` | integer | — | Last line to select (inclusive). If omitted, only `start` line is selected |

**Example:**

````markdown
```editor:select-lines-in-range
file: ~/exercises/app.py
start: 5
stop: 10
```
````

## editor:replace-text-selection

Replaces the currently selected text in a file (selected via `editor:select-matching-text` or `editor:select-lines-in-range`).

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `text` | string | (required) | Replacement text |

**Example:**

````markdown
```editor:replace-text-selection
file: ~/exercises/deployment.yaml
text: nginx:latest
```
````

### Two-step workflow for multi-line replacements

When replacing a multi-line block of text, use `editor:select-matching-text` to highlight the target block first, then `editor:replace-text-selection` to apply the replacement. Place explanatory commentary between the two actions so the learner has time to read the highlighted code and understand what is about to change. This is preferred over a single `editor:replace-matching-text` for multi-line changes because the instant replacement otherwise makes it difficult for the learner to see what was there before.

**Example — select, explain, then replace a multi-line block:**

First, highlight the block that will be replaced:

````markdown
```editor:select-matching-text
file: ~/exercises/deployment.yaml
text: |
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
```
````

The current resource requests are quite conservative. We are going to replace them with higher requests and also add resource limits. This ensures the container gets enough resources to run reliably while preventing it from consuming more than its fair share on the cluster.

````markdown
```editor:replace-text-selection
file: ~/exercises/deployment.yaml
text: |
  resources:
    requests:
      cpu: 250m
      memory: 256Mi
    limits:
      cpu: 500m
      memory: 512Mi
```
````

## editor:replace-matching-text

Find and replace text in a single step, without needing to first select and then replace separately. Best suited for **single-line or short replacements** where the change is easy for the learner to follow.

For **multi-line replacements**, prefer the two-step workflow of `editor:select-matching-text` followed by `editor:replace-text-selection`. When a large block is replaced in a single action, the change happens instantly and the learner has no opportunity to see what was there before. The two-step approach highlights the target block first, giving the learner time to read the existing code, and then a separate action applies the replacement. Place explanatory commentary between the two actions so the learner understands what is about to change and why. When the replacement would cover an entire file's contents — for example, swapping one code example for the next in a teaching sequence — consider providing the new version as a separate pre-created file instead, so the learner retains visibility of the original for comparison.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `match` | string | (required) | Text or regex to find |
| `replacement` | string | (required) | Replacement text |
| `isRegex` | boolean | `false` | Treat `match` as regex |
| `group` | integer | — | Regex subgroup to replace |
| `start` | integer | — | Start line for search range |
| `stop` | integer | — | End line for search range (exclusive) |
| `count` | integer | `1` | Number of matches to replace. `-1` = all matches |

**Example — simple replacement:**

````markdown
```editor:replace-matching-text
file: ~/exercises/deployment.yaml
match: "nginx:1.19"
replacement: "nginx:1.25"
```
````

**Example — regex replace all occurrences:**

````markdown
```editor:replace-matching-text
file: ~/exercises/deployment.yaml
match: "replicas: [0-9]+"
replacement: "replicas: 3"
isRegex: true
count: -1
```
````

### Multi-line matching and replacement

The `match` and `replacement` properties can span multiple lines, allowing you to find and replace entire blocks of content such as YAML fragments, code blocks, or configuration sections.

**Example — replace a multi-line YAML block:**

````markdown
```editor:replace-matching-text
file: ~/exercises/deployment.yaml
match: |
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
replacement: |
  resources:
    requests:
      cpu: 250m
      memory: 256Mi
    limits:
      cpu: 500m
      memory: 512Mi
```
````

**YAML indentation indicator for preserving leading spaces:** When the replacement text has leading spaces on the first line (common with indented code or YAML fragments), use the YAML block scalar **indent indicator** — a digit after the `|` that specifies how many spaces form the indentation. Without it, YAML may strip or misinterpret the leading whitespace.

**Example — replacing indented content using indent indicator:**

If the content to match is nested inside a YAML structure and starts with spaces, use `|2` (or the appropriate indent level) to tell the YAML parser that the content is indented by that many spaces:

````markdown
```editor:replace-matching-text
file: ~/exercises/deployment.yaml
match: |2
      containers:
      - name: app
        image: myapp:v1
replacement: |2
      containers:
      - name: app
        image: myapp:v2
        env:
        - name: LOG_LEVEL
          value: debug
```
````

The digit after `|` (e.g., `|2`, `|4`, `|6`) indicates the number of spaces used for indentation in the block content. The YAML parser uses this to correctly determine where the content starts, preserving all leading spaces as part of the actual text. Without the indicator, the YAML parser treats the leading spaces as YAML indentation and strips them, causing the match to fail or the replacement to be inserted with incorrect indentation.

## editor:replace-lines-in-range

Replaces a range of lines with new content.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `start` | integer | (required) | First line to replace (1-indexed, inclusive) |
| `stop` | integer | (required) | Last line to replace (inclusive) |
| `text` | string | (required) | Replacement content |

**Example:**

````markdown
```editor:replace-lines-in-range
file: ~/exercises/app.py
start: 5
stop: 10
text: |
  def new_function():
      return "replaced"
```
````

## editor:delete-lines-in-range

Deletes a range of lines from a file.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `start` | integer | (required) | First line to delete (1-indexed, inclusive) |
| `stop` | integer | — | Last line to delete (inclusive). If omitted, only `start` line is deleted |

**Example:**

````markdown
```editor:delete-lines-in-range
file: ~/exercises/app.py
start: 5
stop: 10
```
````

## editor:delete-matching-lines

Deletes lines matching a string or regex, optionally including surrounding lines.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `match` | string | (required) | Text or regex to match |
| `isRegex` | boolean | `false` | Treat `match` as regex |
| `before` | integer | `0` | Lines before match to also delete. `-1` = all lines before |
| `after` | integer | `0` | Lines after match to also delete. `-1` = all lines after |

**Example:**

````markdown
```editor:delete-matching-lines
file: ~/exercises/app.py
match: "# TODO: remove this"
after: 1
```
````

## editor:copy-file

Copies a file to a new location.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `src` | string | (required) | Source file path |
| `dest` | string | (required) | Destination file path |
| `open` | boolean | `true` | Open the destination file in editor after copying |

**Example:**

````markdown
```editor:copy-file
src: ~/exercises/template.yaml
dest: ~/exercises/deployment.yaml
```
````

## editor:rename-file

Renames or moves a file.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `src` | string | (required) | Current file path |
| `dest` | string | (required) | New file path |
| `open` | boolean | `true` | Open file in editor after renaming |

**Example:**

````markdown
```editor:rename-file
src: ~/exercises/old-name.yaml
dest: ~/exercises/new-name.yaml
```
````

## editor:close-file

Closes a file tab in the editor. No-op if the file is not open.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |

**Example:**

````markdown
```editor:close-file
file: ~/exercises/deployment.yaml
```
````

## editor:delete-file

Deletes a file from the filesystem. If open in the editor, it will be closed first.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |

**Example:**

````markdown
```editor:delete-file
file: ~/exercises/old-config.yaml
```
````
