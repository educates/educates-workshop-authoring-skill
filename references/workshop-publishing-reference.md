# Workshop Publishing Reference

This document describes how to publish a standalone Educates workshop to the GitHub container registry (GHCR) using a GitHub Actions workflow.

Publishing a workshop creates an OCI image artefact containing the workshop content files and pushes it to GHCR. It also creates a GitHub release with Kubernetes resource files attached as assets for deploying the workshop to an Educates installation.

## When to Use This

This publishing approach applies only to standalone workshops that live in their own Git repository. If the workshop is part of a course containing multiple workshops, do not use this GitHub action — publishing for courses is handled differently.

## GitHub Actions Workflow

Create a `.github/workflows/publish-workshop.yaml` file in the workshop repository with the following content:

```yaml
name: Publish Workshop

on:
  push:
    tags:
      - "[0-9]+.[0-9]+"

jobs:
  publish-workshop:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      - name: Create release
        uses: educates/educates-github-actions/publish-workshop@v7
        with:
          token: ${{secrets.GITHUB_TOKEN}}
```

The workflow is triggered when a Git tag matching the pattern `X.Y` (e.g., `1.0`, `2.3`) is pushed to the repository. This means publishing is done by tagging a commit and pushing the tag:

```bash
git tag 1.0
git push --tags
```

## How Publishing Relates to workshop.yaml

The GitHub action uses the `educates publish-workshop` command internally. For this to work, the workshop definition (`resources/workshop.yaml`) must include a `spec.publish` section that defines where the OCI image should be published. This section is already part of the standard workshop.yaml structure generated when creating a workshop:

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
```

The `$(image_repository)` variable is resolved during publishing to the appropriate GHCR path based on the GitHub repository. The `$(workshop_version)` variable is set to the Git tag used to trigger the workflow.

## Controlling Published Files

The `spec.publish.files` section controls which files from the workshop repository are packaged into the OCI image. By default, only the workshop instructions (`/workshop/**`), exercise files (`/exercises/**`), and the README are included. This keeps the published image small by excluding development files like `.github/`, `resources/`, and other repository content.

To customise what is included, adjust the `includePaths` list in `spec.publish.files`:

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
      - /templates/**
      - /README.md
```

## Action Configuration

The following parameters can be set in the `with` clause of the GitHub action:

| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | GitHub access token. Set to `${{secrets.GITHUB_TOKEN}}` or an appropriate personal access token variable reference. |
| `path` | No | Relative directory path under `$GITHUB_WORKSPACE` to the workshop files. Defaults to `.` (the repository root). |
| `trainingportal-resource-file` | No | Relative path under the workshop directory to the `TrainingPortal` resource file. Defaults to `resources/trainingportal.yaml`. |
| `workshop-resource-file` | No | Relative path under the workshop directory to the `Workshop` resource file. Defaults to `resources/workshop.yaml`. |
