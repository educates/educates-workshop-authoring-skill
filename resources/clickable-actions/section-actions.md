# Section Clickable Actions

Actions for creating collapsible sections to organize optional or progressive content in workshop instructions.

## section:begin and section:end

Marks the beginning and end of a collapsible section. Content between these markers is initially hidden and revealed when the user clicks the section header.

### section:begin Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `title` | string | (required) | Text displayed in the section header banner |
| `name` | string | — | Unique identifier. Required when nesting sections so begin/end pairs can be matched |
| `prefix` | string | `"Section"` | Prefix shown before the title (displayed as "Prefix: Title") |
| `description` | string | — | Additional text shown below the title (pre-formatted) |
| `open` | boolean | `false` | Set to `true` to have the section expanded by default |
| `hidden` | boolean | `false` | Hide the section header. Useful with `autostart`/`cascade` for automatic content reveal |

### section:end Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `name` | string | — | Must match the corresponding `section:begin` name |
| `cascade` | boolean | `false` | Continue cascade to the next action after the section end |
| `toggle` | boolean | `true` | Set to `false` to prevent auto-collapsing when cascade passes through |

**Example — basic collapsible section:**

````markdown
```section:begin
title: Bonus Exercise
```

This optional exercise explores advanced configuration...

```section:end
```
````

**Example — nested sections with names:**

````markdown
```section:begin
name: questions
title: Questions
```

```section:begin
name: question-1
prefix: Question
title: "1"
```

What command lists running pods?

```section:end
name: question-1
```

```section:end
name: questions
```
````

**Example — section expanded by default:**

````markdown
```section:begin
title: Prerequisites
open: true
```

Ensure you have completed the setup steps...

```section:end
```
````

**Example — auto-run command and collapse section:**

````markdown
```section:begin
name: setup
title: Run Setup
```

```terminal:execute
command: ./setup.sh
cascade: true
```

```section:end
name: setup
```
````

When the user expands the section, the command runs. When the command completes, the `cascade` triggers `section:end`, which collapses the section.

## section:heading

Displays a non-collapsible heading styled like a clickable action. Clicking marks it as completed but triggers no other action.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `title` | string | (required) | Heading text |
| `prefix` | string | — | Prefix shown before the title |

**Example:**

````markdown
```section:heading
title: Review Questions
```
````
