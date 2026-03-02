# Workshop Image Reference

The workshop container image defines the base environment available to users during a workshop session. This includes the operating system, pre-installed tools, language runtimes, and utilities.

## Default Image

By default, workshops use the `base-environment` image. This image is suitable for most workshops and does not need to be specified explicitly — if no `workshop.image` field is set in the workshop definition, Educates uses the `base-environment` image matched to the current version of the Educates operator.

Most generated workshops should rely on this default. Only specify a different image when the workshop has a specific requirement that the base environment cannot satisfy.

## Available Workshop Images

Educates provides several pre-built workshop images. These can be referenced by short name in the `workshop.image` field, with `:*` as the version tag to automatically match the current Educates operator version:

| Short name | Use case |
|---|---|
| `base-environment:*` | General-purpose workshops. Includes standard OS tools and a Python distribution. This is the default. |
| `jdk8-environment:*` | Workshops requiring Java 8 (JDK). |
| `jdk11-environment:*` | Workshops requiring Java 11 (JDK). |
| `jdk17-environment:*` | Workshops requiring Java 17 (JDK). |
| `jdk21-environment:*` | Workshops requiring Java 21 (JDK). |
| `conda-environment:*` | Workshops requiring the Anaconda Python distribution instead of the standard system Python. |

### When to choose a non-default image

- **Java workshops**: Use the appropriate `jdk*-environment` image matching the Java version the workshop requires. Pick the JDK version that matches the application or framework being taught.
- **Anaconda Python**: Use `conda-environment` only when the workshop specifically needs the Anaconda distribution (e.g., for scientific computing packages, conda package management, or Jupyter notebook environments that rely on conda). The standard Python included in `base-environment` is sufficient for general Python workshops.

## Specifying the Image in workshop.yaml

When a non-default image is needed, add the `workshop.image` field under `spec.workshop`:

```yaml
spec:
  workshop:
    image: jdk17-environment:*
    files:
    - image:
        url: "$(image_repository)/{workshop-name}-files:$(workshop_version)"
```

The `workshop.image` field sits alongside the existing `workshop.files` configuration. The rest of the workshop definition remains the same. No `includePaths` is needed here because the published OCI image is already filtered via `spec.publish.files`.

## Custom Workshop Images

Building a fully custom workshop image is an advanced workflow. Do not use a custom image unless the user explicitly requests it. When a custom image is required, the `workshop.image` field should reference the image location:

```yaml
spec:
  workshop:
    image: $(image_repository)/{workshop-name}-image:$(workshop_version)
    files:
    - image:
        url: "$(image_repository)/{workshop-name}-files:$(workshop_version)"
```

The `$(image_repository)` and `$(workshop_version)` data variables follow the same convention as `workshop.files` — they are rewritten during publishing. The `-image` suffix distinguishes the custom workshop base image from the `-files` OCI artefact containing workshop content.

### Image pull policy

Educates sets the image pull policy to `Always` for the version tags `:main`, `:master`, `:develop`, and `:latest`. All other tags are treated as immutable and cached on Kubernetes nodes after the initial pull.

## Deprecated Field

In older versions of Educates, the workshop image could be specified using `content.image`. This is deprecated — always use `workshop.image` instead.
