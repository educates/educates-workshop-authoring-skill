# Editor Terminal Clickable Actions

Actions for managing terminal sessions within the VS Code editor. These are separate from the dashboard `terminal:*` actions which manage terminals on the dashboard terminals tab.

## editor:open-terminal

Opens or creates a terminal within the VS Code editor.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `session` | string | `"educates"` | Terminal session name. If a terminal with this name exists, it will be shown; otherwise a new one is created |

**Example:**

````markdown
```editor:open-terminal
session: build
```
````

## editor:send-to-terminal

Sends text or a command to a terminal within the VS Code editor. If the terminal does not exist, it will be created.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `text` | string | (required) | Text to send |
| `session` | string | `"educates"` | Terminal session name |
| `endl` | boolean | `true` | Append newline (executes as command). Set `false` for partial input |

**Example — execute a command:**

````markdown
```editor:send-to-terminal
text: npm run build
session: build
```
````

**Example — send input without newline:**

````markdown
```editor:send-to-terminal
text: yes
session: build
endl: false
```
````

## editor:interrupt-terminal

Sends Ctrl+C to interrupt a running command in a VS Code editor terminal.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `session` | string | `"educates"` | Terminal session name |

**Example:**

````markdown
```editor:interrupt-terminal
session: build
```
````

## editor:clear-terminal

Clears the terminal buffer of a VS Code editor terminal.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `session` | string | `"educates"` | Terminal session name |

**Example:**

````markdown
```editor:clear-terminal
session: build
```
````

## editor:close-terminal

Closes and disposes of a VS Code editor terminal.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `session` | string | `"educates"` | Terminal session name |

**Example:**

````markdown
```editor:close-terminal
session: build
```
````

## editor:execute-command

Executes a registered VS Code command.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `command` | string | (required) | VS Code command identifier |
| `args` | array | — | Arguments to pass to the command |

**Example:**

````markdown
```editor:execute-command
command: spring.initializr.maven-project
args:
- language: Java
  dependencies: [ "actuator", "webflux" ]
  artifactId: demo
  groupId: com.example
```
````
