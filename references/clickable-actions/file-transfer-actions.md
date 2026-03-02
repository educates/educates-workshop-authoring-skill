# File Transfer Clickable Actions

Actions for downloading files from and uploading files to the workshop session. File transfers must be enabled in the workshop configuration by setting `spec.session.applications.files.enabled: true` in `resources/workshop.yaml`. See [workshop-yaml-reference.md](../workshop-yaml-reference.md) for the full workshop definition structure.

## files:download-file

Triggers saving a file from the workshop session to the user's local computer.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `path` | string | — | File path relative to the home directory (or overridden files directory). Use `path` or `url`, not both |
| `url` | string | — | URL to download from an alternate backend service. Hostname must share the same parent domain as the ingress domain |
| `download` | string | — | Local filename to save as (basename only, no directory path). Defaults to basename of `path` |
| `preview` | boolean | `false` | Show file contents in the code block as a preview. Not recommended for large files |

**Example — simple download:**

````markdown
```files:download-file
path: .kube/config
```
````

**Example — download with custom filename and preview:**

````markdown
```files:download-file
path: .kube/config
download: kubeconfig.yaml
preview: true
```
````

## files:copy-file

Copies the contents of a file from the workshop session to the user's browser clipboard (paste buffer) instead of downloading it.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `path` | string | — | File path relative to the home directory |
| `url` | string | — | URL to copy from an alternate backend service |
| `preview` | boolean | `false` | Show file contents in the code block |

**Example:**

````markdown
```files:copy-file
path: .kube/config
preview: true
```
````

## files:upload-file

Presents a file picker to upload a single file with a predetermined name.

**Properties:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `path` | string | (required) | Filename the uploaded file will be saved as in the uploads directory |

**Example:**

````markdown
```files:upload-file
path: kubeconfig.yaml
```
````

## files:upload-files

Presents a file picker to upload one or more files. Uploaded files keep their original names.

**Example:**

````markdown
```files:upload-files
```
````
