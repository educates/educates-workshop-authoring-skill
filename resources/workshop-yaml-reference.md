# Workshop Definition Reference (workshop.yaml)

This document describes the structure and configuration options for the Educates `workshop.yaml` file.

## Document Structure Overview

The following skeleton shows the complete structure with full paths. Use this as a reference when reading snippets later in this document:

```yaml
apiVersion: training.educates.dev/v1beta1
kind: Workshop
metadata:
  name: ""                              # metadata.name (REQUIRED)
spec:
  title: ""                             # spec.title (REQUIRED)
  description: ""                       # spec.description (REQUIRED)
  duration: ""                          # spec.duration (REQUIRED)
  difficulty: ""                        # spec.difficulty (REQUIRED)
  labels: []                            # spec.labels
  publish:                              # spec.publish (REQUIRED)
    image: ""                           # spec.publish.image
    files: []                           # spec.publish.files
  workshop:                             # spec.workshop (REQUIRED)
    image: ""                           # spec.workshop.image (optional, defaults to base-environment)
    files: []                           # spec.workshop.files
  session:                              # spec.session
    namespaces:                         # spec.session.namespaces
      budget: ""                        # spec.session.namespaces.budget
      security:                         # spec.session.namespaces.security
        token:                          # spec.session.namespaces.security.token
          enabled: false                # spec.session.namespaces.security.token.enabled
    applications:                       # spec.session.applications
      terminal:                         # spec.session.applications.terminal
        enabled: true
        layout: split
      editor:                           # spec.session.applications.editor
        enabled: true
      console:                          # spec.session.applications.console
        enabled: true
      docker:                           # spec.session.applications.docker
        enabled: true
        storage: ""                     # spec.session.applications.docker.storage (default 5Gi)
        memory: ""                      # spec.session.applications.docker.memory (default 768Mi)
        socket:                         # spec.session.applications.docker.socket
          enabled: true
        compose:                        # spec.session.applications.docker.compose
          services: {}
      registry:                         # spec.session.applications.registry
        enabled: true
        storage: ""                     # spec.session.applications.registry.storage (default 5Gi)
        memory: ""                      # spec.session.applications.registry.memory (default 768Mi)
      vcluster:                         # spec.session.applications.vcluster
        enabled: true
        version: ""                     # spec.session.applications.vcluster.version
        resources:                      # spec.session.applications.vcluster.resources
          syncer:
            memory: ""
        ingress:                        # spec.session.applications.vcluster.ingress
          enabled: true
          subdomains: []
        objects: []                     # spec.session.applications.vcluster.objects
        services:                       # spec.session.applications.vcluster.services
          fromVirtual: []
          fromHost: []
      git:                              # spec.session.applications.git
        enabled: true
      slides:                           # spec.session.applications.slides
        enabled: true
        reveal.js:                      # spec.session.applications.slides.reveal.js
          version: ""
        impress.js:                     # spec.session.applications.slides.impress.js
          version: ""
      examiner:                          # spec.session.applications.examiner
        enabled: true
      files:                             # spec.session.applications.files
        enabled: true
    objects: []                         # spec.session.objects
    resources:                          # spec.session.resources
      memory: ""                        # spec.session.resources.memory
      storage: ""                       # spec.session.resources.storage
    ingresses: []                       # spec.session.ingresses
    dashboards: []                      # spec.session.dashboards
  environment:                          # spec.environment
    objects: []                         # spec.environment.objects
  request:                              # spec.request
    objects: []                         # spec.request.objects
```

**Note:** Snippets throughout this document show partial YAML. Each snippet includes a path comment indicating where it belongs in the overall structure.

## Base Template

```yaml
apiVersion: training.educates.dev/v1beta1
kind: Workshop
metadata:
  name: "{workshop-name}"
spec:
  title: "{workshop-title}"
  description: "{workshop-description}"
  duration: 30m
  difficulty: beginner
  publish:
    image: "$(image_repository)/{workshop-name}-files:$(workshop_version)"
    files:
    - directory:
        path: .
      includePaths:
      - /workshop/**
      - /exercises/**
      - /README.md
  workshop:
    files:
    - image:
        url: "$(image_repository)/{workshop-name}-files:$(workshop_version)"
  session:
    namespaces:
      budget: medium
      security:
        token:
          enabled: false
    applications:
      terminal:
        enabled: true
        layout: split
      editor:
        enabled: true
```

## Required Fields

| Field | Description |
|-------|-------------|
| `metadata.name` | Machine-readable identifier (lowercase, dashes allowed, max 25 chars, recommend `lab-` prefix) |
| `spec.title` | Human-readable workshop title |
| `spec.description` | One to two sentence description |
| `spec.duration` | Estimated completion time (e.g., `15m`, `30m`, `1h`, `2h`) |
| `spec.difficulty` | Target audience: `beginner`, `intermediate`, `advanced`, or `extreme` |

## Workshop Files Configuration (CRITICAL)

**ALWAYS use this exact structure for workshop file publishing:**

```yaml
spec:
  publish:
    image: "$(image_repository)/{workshop-name}-files:$(workshop_version)"
    files:
    - directory:
        path: .
      includePaths:
      - /workshop/**
      - /exercises/**
      - /README.md
  workshop:
    files:
    - image:
        url: "$(image_repository)/{workshop-name}-files:$(workshop_version)"
```

Replace `{workshop-name}` with the actual workshop name from `metadata.name`.

The `spec.publish.files` section controls which files from the project directory are packaged into the published OCI image. Only the paths matching `includePaths` are included, keeping the published image small and free of development-time files. The `spec.workshop.files` section specifies where to pull the files from at runtime — since the published image already contains only the needed files, no `includePaths` is required there.

The `/exercises/**` path is included because the `exercises/` directory receives special treatment: when it exists in the imported workshop files, terminals start with `~/exercises` as the working directory and the VS Code editor opens on it instead of the home directory. See the "exercises Directory" section in the main skill document for details.

**IMPORTANT:**
- The `$(image_repository)` and `$(workshop_version)` are variables that MUST be used exactly as shown
- These variables are required for local workshop publishing and deployment workflows
- **NEVER use `spec.content.files`** — this is a deprecated format and must not be used

### Alternative File Sources

The primary source for workshop files is an OCI image (shown above), which is the standard for published workshops. When using the standard OCI image source, `includePaths` is not needed on `spec.workshop.files` because the image is already filtered at publish time via `spec.publish.files`.

The `spec.workshop.files` array also supports Git repository and HTTP sources, and can contain multiple entries that are overlaid in order. For these alternative sources, `includePaths` can be used on `spec.workshop.files` entries to filter what is pulled from the unfiltered source.

**Git repository source:**

```yaml
spec:
  workshop:
    files:
    - git:
        url: https://github.com/organization/repository
        ref: origin/main
      includePaths:
      - /workshop/**
      - /exercises/**
      - /README.md
```

**HTTP archive source:**

```yaml
spec:
  workshop:
    files:
    - http:
        url: https://example.com/workshop-content.tar.gz
      includePaths:
      - /workshop/**
```

**Multiple sources (overlay):**

When the `files` array contains multiple entries, files are downloaded and overlaid in array order — later entries overwrite files from earlier ones at the same path. This is useful for composing a workshop from a base set of files plus customizations:

```yaml
spec:
  workshop:
    files:
    - image:
        url: "$(image_repository)/base-content:$(workshop_version)"
      includePaths:
      - /exercises/**
    - image:
        url: "$(image_repository)/{workshop-name}-files:$(workshop_version)"
      includePaths:
      - /workshop/**
      - /README.md
```

**Common options for all source types:**

| Option | Description |
|--------|-------------|
| `includePaths` | Array of glob patterns for files to include |
| `excludePaths` | Array of glob patterns for files to exclude |
| `newRootPath` | Directory within the source to use as the root (strip a prefix) |

For most workshops, the standard single OCI image source shown at the top of this section is sufficient.

## Session Applications

Ask the user which tools the workshop requires. Include only the applications that are needed.

All application settings belong under `spec.session.applications`.

### Terminal

The terminal is enabled by default and does not need to be explicitly enabled. However, it is recommended to include the configuration for clarity.

```yaml
# Path: spec.session.applications
applications:
  terminal:
    enabled: true
    layout: split
```

- **Default behavior**: Always enabled, even if omitted
- **Recommendation**: Always include `enabled: true` explicitly for clarity
- **Layout options**:
  - `default` - Single terminal
  - `split` - Two terminals stacked above each other in ratio 60/40 (recommended default)
  - `split/2` - Three terminals stacked above each other in ratio 50/25/25
  - `lower` - Single terminal placed below dashboard tabs rather than as a tab, ratio 70/30
  - `none` - No terminal displayed, but can still be created from drop down menu

### Editor

For viewing or editing files in the browser:

```yaml
# Path: spec.session.applications
applications:
  editor:
    enabled: true
```

- **When to enable**: Workshops involving code editing, file creation, or viewing source files

### Kubernetes Console

For Kubernetes cluster visualization:

```yaml
# Path: spec.session.applications
applications:
  console:
    enabled: true
```

- **When to enable**: Workshops teaching Kubernetes concepts where users benefit from a visual cluster view

### Docker Daemon

For building and running containers. Each workshop session gets its own docker daemon running as a sidecar container.

```yaml
# Path: spec.session.applications
applications:
  docker:
    enabled: true
```

- **When to enable**: Workshops involving container building, Docker commands, or container runtime exercises
- **Security**: Requires a privileged container for the docker daemon. For workshops with untrusted users, host them in a disposable Kubernetes cluster that is destroyed after the workshop.

**Storage and memory:**

The docker daemon mounts a persistent volume for storing pulled and built images. Defaults are 5Gi storage and 768Mi memory:

```yaml
# Path: spec.session.applications
applications:
  docker:
    enabled: true
    storage: 20Gi
    memory: 1Gi
```

**Docker Compose services:**

To auto-start services when the workshop session begins, provide `docker compose` compatible configuration under `compose.services`. Ports must be explicitly exposed to `127.0.0.1` to be accessible from the workshop container:

```yaml
# Path: spec.session.applications
applications:
  docker:
    enabled: true
    compose:
      services:
        grafana-workshop:
          image: grafana/grafana:7.1.3
          ports:
          - "127.0.0.1:3000:3000"
          environment:
          - GF_AUTH_ANONYMOUS_ENABLED=true
```

If a compose service needs access to workshop files, mount the `workshop` named volume:

```yaml
# Path: spec.session.applications.docker.compose.services
        my-service:
          volumes:
          - type: volume
            source: workshop
            target: /mnt
```

**Socket control:**

When compose services are defined, the docker socket is **not** exposed to the workshop container by default. To re-enable it (e.g., so users can run `docker` commands alongside compose services):

```yaml
# Path: spec.session.applications
applications:
  docker:
    enabled: true
    socket:
      enabled: true
```

Tools that need the socket location programmatically should use the `DOCKER_HOST` environment variable available in the workshop terminal.

### Image Registry

A separate per-session container image registry for pushing and pulling images built during the workshop using tools such as `docker build`, `kpack`, or `kaniko`.

```yaml
# Path: spec.session.applications
applications:
  registry:
    enabled: true
```

- **When to enable**: Workshops where users need to push images to a registry (often paired with Docker)
- **Secure ingress required**: The registry is only fully usable when Educates is deployed with secure ingress, since an insecure registry would not be trusted by the Kubernetes cluster for deployments.

**Storage and memory:**

The registry mounts a persistent volume for storing images. Defaults are 5Gi storage and 768Mi memory:

```yaml
# Path: spec.session.applications
applications:
  registry:
    enabled: true
    storage: 20Gi
    memory: 1Gi
```

**Automatic credential injection:**

- `$HOME/.docker/config.json` is injected into the workshop session so tools like `docker` authenticate automatically.
- A Kubernetes secret of type `kubernetes.io/dockerconfigjson` is created in the session namespace and applied to the `default` service account, so deployments using the default service account can pull images without additional configuration.

**Data variables:** When the registry is enabled, variables such as `registry_host`, `registry_username`, `registry_password`, `registry_auth_token`, `registry_secret`, and `registry_auth_file` become available. See the [Data Variables Reference](data-variables-reference.md) for the full list.

### Virtual Cluster (vcluster)

Provisions a virtual cluster giving the workshop user the appearance of a full Kubernetes cluster with cluster-admin access, running inside the session namespace of the host cluster.

```yaml
# Path: spec.session.applications
applications:
  vcluster:
    enabled: true
```

- **When to enable**: Workshops requiring cluster-admin operations, installing operators, creating namespaces, or other actions that need elevated Kubernetes privileges

**Kubernetes version:**

The virtual cluster defaults to the latest supported Kubernetes version. To pin a specific version:

```yaml
# Path: spec.session.applications
applications:
  vcluster:
    enabled: true
    version: "1.27"
```

**Security policy:**

The default security policy for workloads in the virtual cluster is `baseline` (allows running as root, binding system ports, etc.). To restrict it:

```yaml
# Path: spec.session
session:
  namespaces:
    security:
      policy: restricted
```

**Resource budget and syncer memory:**

Resource quotas and limit ranges from the session namespace budget apply to the virtual cluster. The budget must accommodate CoreDNS which is always deployed in the virtual cluster. Virtual cluster control plane services run in a separate namespace and default to 1Gi memory for the syncer. To override:

```yaml
# Path: spec.session.applications
applications:
  vcluster:
    enabled: true
    resources:
      syncer:
        memory: 768Mi
```

**Ingress:**

Ingress resources created in the virtual cluster are automatically synced to the host cluster. For advanced features, enable the Contour ingress controller:

```yaml
# Path: spec.session.applications
applications:
  vcluster:
    enabled: true
    ingress:
      enabled: true
      subdomains:
      - default
```

Ingress hostnames must be within `$(session_name).$(ingress_domain)` or `{subdomain}.$(session_name).$(ingress_domain)`.

**Deploying resources into the virtual cluster:**

Use `vcluster.objects` to deploy resources into the virtual cluster at session creation. Namespaced resources must include `namespace`. Session variables are substituted:

```yaml
# Path: spec.session.applications
applications:
  vcluster:
    enabled: true
    objects:
    - apiVersion: v1
      kind: ConfigMap
      metadata:
        name: session-details
        namespace: default
      data:
        INGRESS_DOMAIN: "$(ingress_domain)"
```

For more complex deployments (e.g., installing kapp-controller or operators), add `App` resources to `session.objects` targeting the `$(vcluster_namespace)` namespace with `$(vcluster_secret)` for kubeconfig access.

**Service mapping:**

Map services between the virtual cluster and the host cluster:

```yaml
# Path: spec.session.applications
applications:
  vcluster:
    enabled: true
    services:
      fromVirtual:
      - from: my-virtual-namespace/my-virtual-service
        to: my-host-service
      fromHost:
      - from: $(workshop_namespace)/my-host-service
        to: my-virtual-namespace/my-virtual-service
```

**Data variables:** `vcluster_namespace` (namespace of the virtual cluster control plane) and `vcluster_secret` (name of the kubeconfig secret) are available as session data variables. See the [Data Variables Reference](data-variables-reference.md).

### Examiner

For running verification tests that check user progress:

```yaml
# Path: spec.session.applications
applications:
  examiner:
    enabled: true
```

- **When to enable**: Workshops that include verification steps where users click to check whether they completed a task correctly
- See [clickable-actions/examiner-actions.md](clickable-actions/examiner-actions.md) for how to write test scripts and use the `examiner:execute-test` clickable action

### File Downloads and Uploads

For allowing users to download files from or upload files to the workshop session:

```yaml
# Path: spec.session.applications
applications:
  files:
    enabled: true
```

- **When to enable**: Workshops where users need to download generated files (e.g., kubeconfig, certificates) to their local machine or upload files from their local machine into the session
- See [clickable-actions/file-transfer-actions.md](clickable-actions/file-transfer-actions.md) for the `files:download-file`, `files:copy-file`, `files:upload-file`, and `files:upload-files` clickable actions

### Git Server

A local Git server hosted from the workshop container. Each session gets its own Git server instance with unique credentials, automatically deleted when the session terminates.

```yaml
# Path: spec.session.applications
applications:
  git:
    enabled: true
```

- **When to enable**: Workshops involving CI/CD pipelines, source code modification and push, or any scenario where users need their own Git repository without requiring accounts on hosted services like GitHub or GitLab

**How it works:**

The Git server supports any number of repositories. For use in the terminal, the following environment variables are set: `GIT_PROTOCOL`, `GIT_HOST`, `GIT_USERNAME`, `GIT_PASSWORD`. In workshop instructions the same variable names in lower case can be used as data variables.

Git credentials are pre-configured in the workshop user account, so `git clone` and `git push` work without supplying credentials. Users only need the credentials when configuring external systems (e.g., adding them to a CI/CD pipeline).

**Cloning and pushing:**

```bash
git clone $GIT_PROTOCOL://$GIT_HOST/project.git
cd project
# make changes
git add . && git commit -m "Changes" && git push
```

**Pre-creating repositories from setup scripts:**

Repositories can be pre-created from a `setup.d` script. When cloning from a remote server, the `--bare` flag is required because only the bare repository (normally inside `.git`) is needed:

```bash
#!/bin/bash
set -eo pipefail
cd /opt/git/repositories
git clone --bare https://github.com/example/project.git
```

**Webhook hooks:**

To fire a webhook when changes are pushed, provide an executable `post-receive` hook script in the `hooks` directory of the bare repository under `/opt/git/repositories`.

**Data variables:** See the [Data Variables Reference](data-variables-reference.md) for `git_protocol`, `git_host`, `git_username`, and `git_password`.

### Presentation Slides

Serves static slide content from the `workshop/slides/` directory via a built-in HTTP server and automatically creates a dashboard tab for the slides.

```yaml
# Path: spec.session.applications
applications:
  slides:
    enabled: true
```

- **When to enable**: Workshops that include a presentation alongside the hands-on instructions

**How it works:**

Place slide files in the `workshop/slides/` directory. The default web page must be `index.html`. Anything in the directory is served as static files.

**reveal.js support:**

Static assets for reveal.js versions 3.X and 4.X are bundled. Specify the version to use:

```yaml
# Path: spec.session.applications
applications:
  slides:
    enabled: true
    reveal.js:
      version: "4.X"
```

The version can be an exact version or a semver-style range selector.

**impress.js support:**

Static assets for impress.js version 1.X are bundled:

```yaml
# Path: spec.session.applications
applications:
  slides:
    enabled: true
    impress.js:
      version: "1.X"
```

**PDF slides:**

For PDF-based slides, add the PDF file to `workshop/slides/` and create an `index.html` that embeds the PDF.

**Linking to specific slides:**

If using reveal.js with history enabled or section IDs, link to a specific slide from workshop instructions using `/slides/#/section-name`. When the workshop content is displayed in the dashboard, slide links open in the slides tab rather than a separate browser window.

## Kubernetes Access

Kubernetes access via security token is enabled by default for historical reasons. Explicitly disable it unless the workshop requires it.

**When workshop does NOT need Kubernetes access:**

```yaml
# Path: spec.session
session:
  namespaces:
    budget: medium
    security:
      token:
        enabled: false
```

**When workshop DOES need Kubernetes access:**

```yaml
# Path: spec.session
session:
  namespaces:
    budget: medium
    security:
      token:
        enabled: true
```

- **Default behavior**: Security token is enabled by default (historical)
- **Recommendation**: Explicitly set `enabled: false` unless the workshop uses `kubectl`, interacts with the Kubernetes API, or uses the Kubernetes console

### ⚠️ Common Error from Old Documentation

**INCORRECT** - Do NOT use this pattern (missing `namespaces` level):

```yaml
# THIS IS INCORRECT - DO NOT USE
# Path: spec.session
session:
  security:
    token:
      enabled: true
```

Old Educates documentation incorrectly showed the `security` configuration directly under `spec.session`. This is wrong and will not work. The `security` configuration MUST be nested under `spec.session.namespaces.security`, not directly under `spec.session`. Always use the correct examples shown above.

## Workshop Labels

Workshop labels are metadata fields used for organizing and filtering workshops. These are distinct from Kubernetes labels in the `metadata` section.

**Correct format** (array of objects with `name` and `value` properties):

```yaml
# Path: spec.labels
labels:
- name: id
  value: educates.dev/lab-markdown-sample
- name: category
  value: getting-started
```

### ⚠️ Common Error from Old Documentation

**INCORRECT** - Do NOT use dictionary format:

```yaml
# THIS IS INCORRECT - DO NOT USE
# Path: spec.labels
labels:
  id: educates.dev/lab-markdown-sample
  category: getting-started
```

Old Educates documentation incorrectly showed labels as a dictionary (key-value pairs). This format is wrong. Always use the array format with objects containing `name` and `value` properties as shown in the correct example above.

## Session Ingresses

Session ingresses use the workshop session proxy to expose an HTTP service for browser access within the workshop. They are configured under `spec.session.ingresses`.

Each entry defines a named route through the session proxy. The proxy creates an externally accessible hostname of the form `{name}-$(session_hostname)` and forwards requests to the specified backend.

### Proxying to a Kubernetes Service

To proxy to a Kubernetes Service running in the session namespace, specify the `host` as the fully qualified service DNS name and the `port`:

```yaml
# Path: spec.session
session:
  ingresses:
  - name: app
    protocol: http
    host: app.$(session_namespace).svc
    port: 8080
```

This routes `app-$(session_hostname)` through the session proxy to `app.$(session_namespace).svc:8080`. See the [Kubernetes Access Reference](kubernetes-access-reference.md) for more details on this variant, including when to prefer it over a manually created Kubernetes Ingress.

### Proxying to a Process in the Workshop Container

To proxy to a process running directly inside the workshop container (e.g., a web application started from the terminal), omit the `host` field. It defaults to `localhost`, routing to the specified port on the workshop container itself:

```yaml
# Path: spec.session
session:
  ingresses:
  - name: app
    protocol: http
    port: 8080
```

This routes `app-$(session_hostname)` through the session proxy to `localhost:8080` inside the workshop container. Use this when the user starts an application process from the terminal that listens on a port and you want to make it accessible in the browser.

### Embedding in a Dashboard Tab

Combine `spec.session.ingresses` with `spec.session.dashboards` to embed a proxied service as a tab in the workshop dashboard:

```yaml
# Path: spec.session
session:
  ingresses:
  - name: app
    protocol: http
    port: 8080
  dashboards:
  - name: App
    url: "$(ingress_protocol)://app-$(session_hostname)/"
```

The `name` in the dashboard entry is the label shown on the tab. The `url` must use `$(ingress_protocol)` to match the protocol Educates is running under (HTTP or HTTPS), and the hostname must match the ingress `name` prefixed to `$(session_hostname)`.

Workshop instructions can then reveal the dashboard tab at the appropriate point using the `dashboard:open-dashboard` clickable action:

````markdown
```dashboard:open-dashboard
name: App
```
````

**Why use session ingresses instead of direct URLs?** The session proxy provides automatic HTTPS when Educates is deployed with secure ingress, avoids mixed-content errors when embedding in iframes, and gates access through the same authentication as the workshop dashboard.

**IMPORTANT:** If you define a session ingress with a `name` of `app`, you cannot also have the user create a separate Kubernetes Ingress using a hostname of `app-$(session_hostname)`. The session proxy has already claimed that hostname.

## Pre-created Resources and Container Configuration

Educates can automatically create Kubernetes resources when a session starts and configure the workshop container's own resource limits. For detailed guidance, examples, and all available options, see [session-objects-reference.md](session-objects-reference.md).

**Summary of available sections:**

| Configuration | Purpose |
|---------------|---------|
| `spec.session.objects` | Kubernetes resources created per session (Deployments, Services, ConfigMaps, etc.) |
| `spec.environment.objects` | Shared resources created once per workshop environment |
| `spec.request.objects` | Resources created when a session is assigned to a user (has access to user identity) |
| `spec.session.resources` | Workshop container memory and storage configuration |

## Decision Flowchart

When generating a workshop.yaml, determine:

1. **What is the workshop topic?** → Sets title and description
2. **Terminal configuration** → Always include with `enabled: true` and `layout: split` (terminal is enabled by default, but include explicitly for clarity)
3. **Does it involve editing code?** → Enable editor
4. **Does it need Kubernetes access?** → If yes, keep security token enabled; if no, explicitly disable with `enabled: false`
5. **Does it teach Kubernetes?** → Enable console (and keep security token enabled)
6. **Does it involve containers?** → Enable docker and possibly registry
7. **Does it need cluster-admin access?** → Enable vcluster
8. **Does it need a local Git repository?** → Enable git (e.g., for CI/CD pipeline workshops)
9. **Does it include a presentation?** → Enable slides and add content to `workshop/slides/`
