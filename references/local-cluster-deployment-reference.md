# Local Cluster Deployment Reference

This document describes how to publish and deploy workshops to a local Educates cluster during development and testing.

All commands described here must be run from within the workshop directory. For a standalone workshop this is the workshop root directory. For a workshop that is part of a course, this is the individual workshop subdirectory (e.g., `workshops/lab-intro-to-kubernetes`).

**Important:** Do not run any of these commands automatically. Only run them when the user explicitly asks to publish, deploy, update, or delete a workshop in the local Educates cluster.

## Local Educates Cluster

These commands apply when a local Educates cluster with a local image registry has been set up using the `educates create-cluster` command.

### Publishing a Workshop

To publish a workshop's content to the local image registry, run:

```bash
educates publish-workshop
```

This packages the workshop content as an OCI image artefact and pushes it to the local image registry associated with the cluster.

### Deploying a Workshop

To deploy a published workshop to the local Educates cluster so it is accessible through the training portal, run:

```bash
educates deploy-workshop
```

The workshop must have been published first before it can be deployed.

### Updating Workshop Content

If changes are made to workshop content (instruction pages, exercise files, or other files included in the published image), the workshop must be republished:

```bash
educates publish-workshop
```

Re-running the publish command updates the content in the local image registry. The next workshop session started will pick up the new content.

### Updating the Workshop Definition

If the workshop definition (`resources/workshop.yaml`) is changed — for example to add a new dashboard tab, enable an application, or modify session resources — run:

```bash
educates update-workshop
```

This updates the workshop configuration in the cluster without needing to republish the content. If both the definition and content have changed, run `educates publish-workshop` first, then `educates update-workshop`.

### Deleting a Workshop

To remove a workshop from the local Educates cluster so it is no longer hosted by the training portal, run:

```bash
educates delete-workshop
```

### Deploying All Workshops in a Course

When working with a course that contains multiple workshops, each workshop must be published and deployed individually from its own subdirectory. When deploying all workshops in a course, deploy them in the order they appear in the course definition. This ensures they are displayed in the correct order in the training portal UI.
