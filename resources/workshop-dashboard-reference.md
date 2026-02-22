# Workshop Dashboard Reference

This document describes the layout and behavior of the Educates workshop dashboard, with guidance for writing workshop instructions that account for the single-tab visibility constraint.

## Dashboard Layout

The workshop dashboard is divided into two main areas:

- **Left panel (~30% width)**: Workshop instructions. Users read through the instructions and interact with clickable actions here.
- **Right panel (~70% width)**: Dashboard tabs. This area displays one tab at a time from a set of available tabs such as Terminal, Editor, Console, and any custom dashboard tabs.

Only one dashboard tab is visible at a time. To see a different tab, the user must click on the tab header or use a `dashboard:open-dashboard` clickable action in the workshop instructions.

## Default Tab on Startup

The Terminal tab is shown by default when a workshop session starts. This is configurable through the workshop definition, but most workshops use this default.

## Switching Between Tabs

Users can switch the visible dashboard tab in two ways:

1. **Manually** — clicking on the tab header in the dashboard area.
2. **Via clickable action** — using `dashboard:open-dashboard` in the workshop instructions to programmatically switch to a named tab.

Example clickable action to switch to the Editor tab:

````markdown
```dashboard:open-dashboard
name: Editor
```
````

### Implicit Tab Exposure by Terminal and Editor Actions

All terminal clickable actions (`terminal:execute`, `terminal:input`, `terminal:interrupt`, `terminal:select`, `terminal:clear`, etc.) automatically expose the Terminal dashboard tab when triggered. Similarly, all editor clickable actions (`editor:open-file`, `editor:create-file`, `editor:replace-matching-text`, `editor:append-lines-to-file`, etc.) automatically expose the Editor dashboard tab.

Because these actions already make the relevant tab visible, do not add a `dashboard:open-dashboard` action immediately after them to open the Terminal or Editor tab — it is redundant.

**Avoid this — redundant tab switch after an editor action:**

````markdown
```editor:replace-matching-text
file: ~/exercises/deployment.yaml
match: "nginx:1.19"
replacement: "nginx:1.25"
```

```dashboard:open-dashboard
name: Editor
```
````

The `dashboard:open-dashboard` above is unnecessary because `editor:replace-matching-text` already exposed the Editor tab.

**Correct — the editor action alone is sufficient:**

````markdown
```editor:replace-matching-text
file: ~/exercises/deployment.yaml
match: "nginx:1.19"
replacement: "nginx:1.25"
```
````

The same applies to terminal actions. For example, `terminal:execute` already exposes the Terminal tab, so following it with `dashboard:open-dashboard` for Terminal is redundant.

Note that `dashboard:open-dashboard` is still needed when switching to other dashboard tabs such as custom web application tabs, Console, or any tab that is not the direct target of a terminal or editor action. The guidance in the next section on managing tab visibility for custom tabs remains important.

## Single-Tab Visibility and Its Implications

Because only one dashboard tab can be seen at a time, workshop authors must be mindful of which tab is visible at each point in the instructions. This is especially important when the workshop uses a custom dashboard tab to display a web application (e.g., accessed through the session proxy).

### The Tab Switching Problem

Consider a workshop where a dashboard tab named "Application" shows a web application. If the instructions then ask the user to run a terminal command (either explicitly via `terminal:execute` or by telling them to type in the terminal), the Terminal tab becomes visible and the Application tab is hidden. The user cannot see the effect of the command on the web application until they switch back to the Application tab.

If the workshop instructions do not account for this, users will be confused — they ran the command but cannot see the result.

### Writing Instructions That Account for Tab Visibility

When workshop instructions involve switching between viewing a web application (or any non-terminal dashboard tab) and running terminal commands, you must explicitly guide the user back to the correct tab to see the result. Follow these patterns:

**Pattern 1: Run command, then switch back to view result**

After a terminal command that affects a web application displayed in a dashboard tab, include a clickable action or instruction to return to that tab:

````markdown
Run the following command to restart the application:

```terminal:execute
command: ./restart-app.sh
```

Once the application has restarted, switch back to the Application tab to see the changes:

```dashboard:open-dashboard
name: Application
```
````

**Pattern 2: Reload the dashboard tab after a command**

If the web application needs to be refreshed to show the result, use `dashboard:reload-dashboard` which both reloads the content and switches focus to the tab:

````markdown
Apply the configuration change:

```terminal:execute
command: ./apply-config.sh
```

Reload the application to see the updated configuration:

```dashboard:reload-dashboard
name: Application
```
````

**Pattern 3: Remind users to switch tabs manually**

When a clickable action is not appropriate (e.g., the user needs to wait for something to complete first), include clear prose telling the user which tab to switch to:

```markdown
After the deployment finishes, click on the **Application** tab in the
dashboard to see the running application.
```

### General Guidelines

- **Track which tab is visible** at each point in the instructions, just as you track the terminal working directory. Every `terminal:execute` action makes the Terminal tab visible. Every `dashboard:open-dashboard` or `dashboard:reload-dashboard` action switches to the named tab.
- **Always guide the user back** to the relevant tab after running a command that changes the visible tab. Do not assume the user will know to switch tabs on their own.
- **Place the tab switch immediately after** the action that requires it. Do not leave multiple steps between running a command and telling the user to look at another tab — they may become confused about what they should be seeing.
- **Consider combining steps** when a command is run only for its side effect on a web application. It may be clearer to run the command and immediately switch to the application tab in quick succession, rather than having the user linger on the terminal output.
