# Session Objects and Resources Reference

Educates can automatically create Kubernetes resources when a workshop session starts and clean them up when the session ends. This is useful for pre-deploying applications, services, or configuration that learners will interact with during the workshop. This document also covers configuring the workshop container's own resources.

## Session Objects (`spec.session.objects`)

Session objects are Kubernetes resources created in the session namespace for each individual workshop session. They are defined in the workshop definition (`resources/workshop.yaml`) and deployed automatically when a session is provisioned.

```yaml
# Path: spec.session
session:
  objects:
  - apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: sample-app
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: sample-app
      template:
        metadata:
          labels:
            app: sample-app
        spec:
          containers:
          - name: app
            image: nginx:latest
            ports:
            - containerPort: 8080
  - apiVersion: v1
    kind: Service
    metadata:
      name: sample-app
    spec:
      selector:
        app: sample-app
      ports:
      - port: 8080
        targetPort: 8080
```

### Key characteristics

- **Automatic namespacing**: Namespaced resources are created in the session namespace without needing to specify `metadata.namespace`.
- **Automatic cleanup**: Owner references link each resource to the `WorkshopSession` custom resource. When the session ends, all session objects are garbage-collected automatically.
- **Data variable substitution**: All session-level data variables are available using `$(variable_name)` syntax. Common examples: `$(session_name)`, `$(session_namespace)`, `$(session_hostname)`, `$(ingress_protocol)`, `$(ingress_domain)`.
- **Cluster-scoped resources**: If a session object is cluster-scoped (e.g., a Namespace or ClusterRole), embed `$(session_name)` in the resource name to ensure uniqueness across concurrent sessions.

### Common patterns

**Pre-deploy an application the learner will interact with:**

```yaml
session:
  objects:
  - apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: frontend
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: frontend
      template:
        metadata:
          labels:
            app: frontend
        spec:
          containers:
          - name: frontend
            image: myregistry/frontend:1.0
            ports:
            - containerPort: 8080
  - apiVersion: v1
    kind: Service
    metadata:
      name: frontend
    spec:
      selector:
        app: frontend
      ports:
      - port: 8080
```

**Create a ConfigMap with session-specific data:**

```yaml
session:
  objects:
  - apiVersion: v1
    kind: ConfigMap
    metadata:
      name: workshop-config
    data:
      session-name: $(session_name)
      session-namespace: $(session_namespace)
```

**Combine with session ingresses to embed a pre-deployed app in the dashboard:**

When session objects deploy an application with a Service, combine `spec.session.objects` with `spec.session.ingresses` and `spec.session.dashboards` to make the application accessible in a dashboard tab:

```yaml
# Path: spec.session
session:
  objects:
  - apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: app
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: app
      template:
        metadata:
          labels:
            app: app
        spec:
          containers:
          - name: app
            image: myregistry/app:1.0
            ports:
            - containerPort: 8080
  - apiVersion: v1
    kind: Service
    metadata:
      name: app
    spec:
      selector:
        app: app
      ports:
      - port: 8080
  ingresses:
  - name: app
    protocol: http
    host: app.$(session_namespace).svc
    port: 8080
  dashboards:
  - name: Application
    url: "$(ingress_protocol)://app-$(session_hostname)/"
```

See [workshop-yaml-reference.md](workshop-yaml-reference.md) for details on session ingresses and dashboards. See [kubernetes-access-reference.md](kubernetes-access-reference.md) for details on accessing Services from the workshop container.

## Environment Objects (`spec.environment.objects`)

Environment objects are Kubernetes resources created once per workshop environment and shared across all sessions. They are deployed in the workshop namespace (not individual session namespaces).

```yaml
# Path: spec.environment
environment:
  objects:
  - apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: shared-service
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: shared-service
      template:
        metadata:
          labels:
            app: shared-service
        spec:
          containers:
          - name: service
            image: myregistry/shared-service:1.0
```

### Key characteristics

- **Created in the workshop namespace**: These resources live alongside the workshop environment, not in individual session namespaces.
- **Shared across sessions**: All sessions in the same workshop environment can access these resources.
- **Automatic cleanup**: Owner references link to the `WorkshopEnvironment` resource for garbage collection.
- **Environment data variables only**: Only environment-level data variables are available for substitution (not session-specific variables like `session_name`). See [data-variables-reference.md](data-variables-reference.md) for the availability columns.

### When to use environment objects

- Shared databases or caches that all workshop sessions connect to
- Common services that would be wasteful to duplicate per session
- Shared container registries or artifact stores

## Request Objects (`spec.request.objects`)

Request objects are Kubernetes resources created when a session is allocated to a specific user (not during initial provisioning). This timing is important for sessions that are pre-provisioned before a user requests them.

```yaml
# Path: spec
request:
  objects:
  - apiVersion: v1
    kind: ConfigMap
    metadata:
      name: user-config
      namespace: $(session_namespace)
    data:
      username: $(username)
      email: $(email)
```

### Key characteristics

- **Created at allocation time**: Unlike session objects (created during provisioning), request objects are created when a user is actually assigned to the session.
- **User data variables available**: In addition to session data variables, request objects can use `$(username)`, `$(first_name)`, `$(last_name)`, and `$(email)` from the training portal. See [data-variables-reference.md](data-variables-reference.md) for details on request data variables.
- **Must specify namespace explicitly**: Unlike session objects, request objects require `metadata.namespace: $(session_namespace)` for namespaced resources.

### When to use request objects

- Resources that need user identity (username, email)
- Resources that should only exist once a real user is assigned, not during pre-provisioning

## Workshop Container Resources (`spec.session.resources`)

The `spec.session.resources` section configures CPU, memory, and storage for the workshop container itself. This is separate from the resource budgets that apply to the session namespace where learners deploy workloads (see [kubernetes-access-reference.md](kubernetes-access-reference.md) for namespace budgets).

### Memory

Override the default memory allocation for the workshop container:

```yaml
# Path: spec.session
session:
  resources:
    memory: 2Gi
```

**Defaults:**
- Without the editor enabled: **512Mi**
- With the editor enabled: **1Gi**

Increase this when the workshop runs memory-intensive processes directly in the workshop container (e.g., builds, local servers, or data processing tasks started from the terminal).

### Persistent Storage

Request persistent storage that survives container restarts:

```yaml
# Path: spec.session
session:
  resources:
    storage: 5Gi
```

This creates a persistent volume claim for the session. Use this when the workshop generates or downloads large files, or when container restarts should not lose user progress.

### Using an Existing Volume

To mount an existing persistent volume claim instead of creating a new one:

```yaml
# Path: spec.session
session:
  resources:
    volume:
      name: $(session_name)-workshop
      subPath: storage
```

## See Also

- [Workshop YAML Reference](workshop-yaml-reference.md) — Complete workshop.yaml structure
- [Data Variables Reference](data-variables-reference.md) — Variables available for substitution in objects
- [Kubernetes Access Reference](kubernetes-access-reference.md) — Namespace budgets, pod security, and ingress patterns
