# Terminal Clickable Actions

Actions for executing commands, sending input, and managing terminal sessions on the dashboard terminals tab.

## terminal:execute

Executes a command in a terminal session.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `command` | string | (required) | The shell command to execute |
| `session` | integer | `1` | Terminal session number |
| `clear` | boolean | `false` | Clear terminal buffer before executing |

**Example — simple command:**

````markdown
```terminal:execute
command: kubectl get pods
```
````

**Example — target a specific terminal session:**

````markdown
```terminal:execute
command: kubectl get pods
session: 2
```
````

**Example — clear terminal before running:**

````markdown
```terminal:execute
command: kubectl get pods
clear: true
```
````

**When to use `clear: true`:** When workshop instructions issue many successive terminal commands — for example, running a series of examples or repeating a command after making changes — output from earlier commands can make it hard to identify the result of the current command. Using `clear: true` clears the terminal buffer before execution so the user only sees the output of that command. This is especially useful when the previous output is not needed for reference.

An alternative to clearing the terminal is to customize the shell prompt so that each command's output is visually separated. For example, colorizing the prompt or adding a blank line before it makes it easier to find where each command starts when scrolling back. See [workshop-setup-reference.md](../workshop-setup-reference.md) for prompt customization via `workshop/profile`.

**Example — command containing YAML-special characters (use block scalar):**

````markdown
```terminal:execute
command: |-
  docker run --rm -p 8080:80 nginx:latest
```
````

**Example — multi-line command:**

````markdown
```terminal:execute
command: |-
  cat << 'EOF' > /tmp/config.yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: example
  EOF
```
````

## terminal:execute-all

Executes a command in all terminal sessions on the terminals tab. The first terminal session is left selected afterward.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `command` | string | (required) | The shell command to execute |
| `clear` | boolean | `false` | Clear terminal buffer before executing |

**Example:**

````markdown
```terminal:execute-all
command: clear
```
````

## terminal:input

Sends text to a terminal as if pasted, without treating it as a command. Useful for responding to interactive prompts (passwords, confirmations, etc.).

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `text` | string | (required) | The text to send |
| `session` | integer | `1` | Terminal session number |
| `endl` | boolean | `true` | Append newline after text. Set to `false` for partial input |

**Example — answering a prompt:**

````markdown
```terminal:input
text: yes
```
````

**Example — sending input without newline:**

````markdown
```terminal:input
text: password123
endl: false
```
````

## terminal:interrupt

Sends Ctrl+C to interrupt a running command in a terminal session.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `session` | integer | `1` | Terminal session number |

**Example:**

````markdown
```terminal:interrupt
session: 1
```
````

## terminal:interrupt-all

Sends Ctrl+C to all terminal sessions.

**Example:**

````markdown
```terminal:interrupt-all
```
````

## terminal:clear

Clears the full terminal buffer (including scrollback) of a terminal session. Has no effect if an application is running in visual mode.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `session` | integer | `1` | Terminal session number |

**Example:**

````markdown
```terminal:clear
session: 1
```
````

## terminal:clear-all

Clears the full terminal buffer of all terminal sessions.

**Example:**

````markdown
```terminal:clear-all
```
````

## terminal:select

Selects a terminal session and gives it focus ready for text entry. Unlike `dashboard:open-dashboard` for the Terminal tab, this ensures the terminal receives keyboard focus.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `session` | integer | `1` | Terminal session number |

**Example:**

````markdown
```terminal:select
session: 2
```
````
