# Local Docker Deployment Reference

Educates workshops are typically deployed to a Kubernetes cluster, but compatible workshops can also be deployed to a local Docker-compatible container runtime for development, testing, or situations where Kubernetes is not available.

A compatible workshop is deployed locally using:

```bash
educates docker workshop deploy
```

This document describes the restrictions that apply to local Docker deployment and the changes needed in the workshop definition and instructions to support it.

## When to Use This

Local Docker compatibility should only be added when specifically requested. By default, workshops target Kubernetes deployment only. If a user asks for a workshop that works on local Docker as well as in Kubernetes, or asks to retrofit local Docker support onto an existing workshop, consult this reference for the required changes.

## Compatibility Restrictions

Not all workshops can run on local Docker. A workshop is incompatible with local Docker deployment if it requires any of the following:

- **Kubernetes access** — workshops that need `kubectl` commands or interact with the Kubernetes API cannot run on local Docker because there is no Kubernetes cluster available.
- **Kubernetes virtual cluster** — the virtual cluster feature (`spec.session.applications.vcluster`) requires a Kubernetes host cluster and cannot be provided in a local Docker environment.
- **Session image registry** — the per-session container image registry (`spec.session.applications.registry`) is a Kubernetes-hosted service and is not available when running under local Docker.
- **Environment, session, or request objects** — the `environment.objects`, `session.objects`, and `request.objects` sections in the workshop definition create Kubernetes resources that cannot be provisioned in a local Docker environment.

If a workshop requires any of these features, it cannot support local Docker deployment.

## Session Proxy URL Port Suffix

The most important change needed for local Docker compatibility involves URLs that access applications through the session proxy.

When a workshop runs in Kubernetes, the workshop dashboard is accessed on standard ports (80 or 443), so session proxy URLs use these same standard ports. When running on local Docker, the dashboard uses a random port number with a `nip.io` address incorporating the IP address of the local machine. The session proxy also uses this non-standard port.

In a Kubernetes-only workshop, session proxy URLs in workshop instructions are typically written as:

```
{{< param ingress_protocol >}}://app-{{< param session_hostname >}}
```

and in the workshop definition (`workshop.yaml`) as:

```
$(ingress_protocol)://app-$(session_hostname)
```

These omit the port because ports 80 and 443 do not need to be specified in URLs. On local Docker, the port is not standard and must be included.

The `ingress_port_suffix` data variable resolves this. When on standard ports (Kubernetes), it is an empty string and has no effect. When on a non-standard port (local Docker), it expands to the port number with a colon prefix (e.g., `:12345`).

For workshops that need to work on both Kubernetes and local Docker, use:

**In workshop instructions (Hugo shortcodes):**

```
{{< param ingress_protocol >}}://app-{{< param session_hostname >}}{{< param ingress_port_suffix >}}
```

**In the workshop definition (data variables):**

```
$(ingress_protocol)://app-$(session_hostname)$(ingress_port_suffix)
```

The `ingress_port_suffix` variable must be appended immediately after `session_hostname` with no space or separator — the variable itself includes the colon when a port is needed.

## Where to Apply the Port Suffix

The port suffix needs to be added wherever a session proxy URL is constructed. Common locations include:

- **Dashboard tab URLs** in `spec.session.dashboards` entries in the workshop definition
- **Clickable actions** in workshop instructions that open or reload dashboard tabs with session proxy URLs (e.g., `dashboard:open-url`, `dashboard:reload-dashboard`)
- **Prose in workshop instructions** that shows or links to session proxy URLs
- **Terminal commands** in `terminal:execute` actions that reference session proxy URLs (e.g., `curl` commands)

## Example: Dashboard Configuration with Port Suffix

A workshop definition that embeds a proxied application as a dashboard tab, compatible with both Kubernetes and local Docker:

```yaml
spec:
  session:
    ingresses:
    - name: app
      port: 8080
    dashboards:
    - name: App
      url: "$(ingress_protocol)://app-$(session_hostname)$(ingress_port_suffix)/"
```

Without the `$(ingress_port_suffix)`, this would work on Kubernetes but fail to load on local Docker because the browser would attempt to connect on port 80 instead of the actual port.

## Retrofitting Local Docker Support

When adding local Docker support to an existing workshop, the main task is to audit all places where session proxy URLs are constructed and append the `ingress_port_suffix` variable. Search for occurrences of `session_hostname` in both the workshop definition (`resources/workshop.yaml`) and all workshop instruction pages (`workshop/content/`) to find every URL that needs updating.

Also verify that the workshop does not use any of the incompatible features listed in the restrictions section above.
