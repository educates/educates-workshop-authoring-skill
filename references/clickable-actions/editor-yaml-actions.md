# Editor YAML Clickable Actions

Structured YAML manipulation actions that preserve comments and handle all YAML styles (block, flow, inline). These replace the deprecated `editor:insert-value-into-yaml` action.

YAML paths use dot notation for mapping keys (`spec.template`), bracket notation with integers for sequence indices (`containers[0]`), bracket notation with key=value for matching sequence items by attribute (`containers[name=nginx]`), and quoted bracket keys for names with dots or special characters (`metadata["labels"]`, `data["index.html"]`).

## editor:set-yaml-value

Sets or updates a value at a specific YAML path. Creates intermediate keys if they do not exist.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `path` | string | (required) | Dot-notation YAML path |
| `value` | any | (required) | Value to set (scalar, mapping, or sequence) |

**Example — set scalar value:**

````markdown
```editor:set-yaml-value
file: ~/exercises/deployment.yaml
path: spec.replicas
value: 3
```
````

**Example — set nested mapping:**

````markdown
```editor:set-yaml-value
file: ~/exercises/deployment.yaml
path: metadata.annotations
value:
  description: "My application"
  version: "1.0"
```
````

## editor:add-yaml-item

Appends an item to the end of a YAML sequence.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `path` | string | (required) | YAML path to the sequence |
| `value` | any | (required) | Item to append |

**Example:**

````markdown
```editor:add-yaml-item
file: ~/exercises/deployment.yaml
path: spec.template.spec.containers
value:
  name: sidecar
  image: busybox:latest
```
````

## editor:insert-yaml-item

Inserts an item at a specific position in a YAML sequence.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `path` | string | (required) | YAML path to the sequence |
| `index` | integer | (required) | Position to insert at (0-indexed) |
| `value` | any | (required) | Item to insert |

**Example:**

````markdown
```editor:insert-yaml-item
file: ~/exercises/deployment.yaml
path: spec.template.spec.containers
index: 0
value:
  name: init
  image: alpine:latest
```
````

## editor:replace-yaml-item

Replaces a specific item in a YAML sequence, identified by index or attribute match.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `path` | string | (required) | YAML path including item selector, e.g. `containers[name=nginx]` or `containers[0]` |
| `value` | any | (required) | Replacement item |

**Example:**

````markdown
```editor:replace-yaml-item
file: ~/exercises/deployment.yaml
path: spec.template.spec.containers[name=nginx]
value:
  name: nginx
  image: nginx:1.25
  ports:
  - containerPort: 8080
```
````

## editor:delete-yaml-value

Deletes a key from a YAML mapping or an item from a sequence.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `path` | string | (required) | YAML path to the key or item to delete |

**Example — delete a mapping key:**

````markdown
```editor:delete-yaml-value
file: ~/exercises/deployment.yaml
path: metadata.labels.app
```
````

**Example — delete a sequence item by attribute match:**

````markdown
```editor:delete-yaml-value
file: ~/exercises/deployment.yaml
path: spec.template.spec.containers[name=sidecar]
```
````

## editor:merge-yaml-values

Merges multiple key-value pairs into an existing YAML mapping.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `path` | string | (required) | YAML path to the mapping |
| `value` | mapping | (required) | Key-value pairs to merge |

**Example:**

````markdown
```editor:merge-yaml-values
file: ~/exercises/deployment.yaml
path: metadata.labels
value:
  app: myapp
  version: v2
  tier: frontend
```
````

## editor:select-yaml-path

Selects (highlights) a YAML node at a specific path in the editor, including both the key and its value.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `file` | string | (required) | File path |
| `path` | string | — | YAML path. If omitted, the entire document is selected |

Note: When workshop commentary describes YAML configuration, use this action to highlight the relevant sections of the YAML file, giving students visual context for what you are discussing.

**Example:**

````markdown
```editor:select-yaml-path
file: ~/exercises/deployment.yaml
path: spec.template.spec.containers
```
````
