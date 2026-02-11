# Dashboard Clickable Actions

Actions for controlling the workshop dashboard, opening URLs, and managing dashboard tabs.

## dashboard:open-url

Opens a URL in a new browser tab or window.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `url` | string | (required) | The URL to open |

**Example:**

````markdown
```dashboard:open-url
url: https://docs.educates.dev
```
````

## dashboard:open-dashboard

Makes a specific dashboard tab visible if it is hidden.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `name` | string | (required) | Dashboard tab name (e.g., `Terminal`, `Console`, `Editor`) |

Note: For terminals, this does not give keyboard focus. Use `terminal:select` instead if you need the terminal to receive keyboard input.

**Example:**

````markdown
```dashboard:open-dashboard
name: Terminal
```
````

## dashboard:create-dashboard

Creates a new dashboard tab with a URL or terminal session.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `name` | string | (required) | Name for the new dashboard tab |
| `url` | string | (required) | URL to embed, or `terminal:<session-name>` for a terminal |

For terminal dashboards, the session name should use lowercase letters, numbers, and hyphens only. Avoid numeric names like "1", "2", "3" as these are reserved for default terminals.

**Example — embed a web URL:**

````markdown
```dashboard:create-dashboard
name: Documentation
url: https://docs.educates.dev
```
````

**Example — create a terminal tab:**

````markdown
```dashboard:create-dashboard
name: Build Terminal
url: terminal:build
```
````

## dashboard:reload-dashboard

Reloads an existing dashboard tab. If the dashboard does not exist, it will be created (making this a safe alternative to `dashboard:create-dashboard` that won't error on duplicates).

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `name` | string | (required) | Dashboard tab name |
| `url` | string | — | New URL to load. Omit to reload the current URL. Cannot change a terminal dashboard's target |
| `focus` | boolean | `true` | Set to `false` to reload without switching focus to the tab |

**Example — reload with new URL:**

````markdown
```dashboard:reload-dashboard
name: App Preview
url: https://myapp-{{< param session_name >}}.{{< param ingress_domain >}}
```
````

**Example — reload without focus:**

````markdown
```dashboard:reload-dashboard
name: App Preview
url: https://www.example.com/
focus: false
```
````

## dashboard:delete-dashboard

Deletes a custom dashboard tab. Cannot delete built-in tabs (Terminal, Console, Editor, Slides). Deleting a terminal dashboard does not destroy the underlying terminal session.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `name` | string | (required) | Dashboard tab name to delete |

**Example:**

````markdown
```dashboard:delete-dashboard
name: Documentation
```
````
