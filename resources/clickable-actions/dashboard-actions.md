# Dashboard Clickable Actions

Actions for controlling the workshop dashboard, opening URLs, and managing dashboard tabs.

For background on the dashboard layout (instructions on the left, a single visible tab on the right) and guidance on writing instructions that account for tab switching, see [Workshop Dashboard Reference](../workshop-dashboard-reference.md).

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

Makes a specific dashboard tab visible if it is hidden. This is the preferred action when you only need to make a tab visible without reloading its content. Use this instead of `dashboard:reload-dashboard` when no content refresh or URL change is needed.

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

When the workshop will later use `dashboard:reload-dashboard` to change the tab's URL (for example, navigating to a sub-path of the same site), prefer using `dashboard:create-dashboard` for the initial setup rather than pre-defining the dashboard in `spec.session.dashboards`. This keeps the initial URL visible in the workshop instructions, making later URL changes easier to follow. See the [Workshop Dashboard Reference](../workshop-dashboard-reference.md) for the full rationale.

## dashboard:reload-dashboard

Reloads an existing dashboard tab. If the dashboard does not exist, it will be created (making this a safe alternative to `dashboard:create-dashboard` that won't error on duplicates).

Because dashboard tabs are iframes with no URL bar, users cannot change the URL themselves. This action with a `url` property is the only way to change what URL a dashboard tab displays.

Do not use `dashboard:reload-dashboard` without a `url` just to make a tab visible — this triggers an unnecessary reload of the iframe content. Use `dashboard:open-dashboard` instead when you only need to switch to the tab.

**Avoid this — unnecessary reload just to make a tab visible:**

````markdown
```dashboard:reload-dashboard
name: Application
```
````

**Correct — use `dashboard:open-dashboard` to switch to a tab without reloading:**

````markdown
```dashboard:open-dashboard
name: Application
```
````

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
url: {{< param ingress_protocol >}}://myapp-{{< param session_hostname >}}
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
