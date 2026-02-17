# Data Variables Reference

Educates provides a set of data variables containing information about the platform installation, the workshop environment, and the individual workshop session. These variables allow workshop content and configuration to be parameterized so that instructions and resources are personalized to each user's session.

Data variables are available in three contexts, each with its own syntax for accessing them.

## Accessing Data Variables

### In Workshop Instructions

Workshop instruction pages use the Hugo `param` shortcode to insert a data variable value:

```markdown
Deploy to the `{{< param session_namespace >}}` namespace.
```

This also works inside clickable actions:

````markdown
```dashboard:open-url
url: {{< param ingress_protocol >}}://app-{{< param session_hostname >}}
```
````

### In the Terminal Shell

Most data variables are available as uppercase environment variables in the workshop terminal:

````markdown
```terminal:execute
command: echo $SESSION_NAMESPACE
```
````

### In the Workshop Definition

In `resources/workshop.yaml`, data variables use the `$(variable_name)` syntax and are expanded automatically when the configuration is applied:

```yaml
# Path: spec.session
session:
  ingresses:
  - name: app
    host: app.$(session_namespace).svc
    port: 8080
```

The set of variables available differs depending on where in the workshop definition they are used. See the availability columns in the table below.

## Core Data Variables

The following table lists the core data variables. The availability columns indicate where each variable can be used:

- **Instructions** — available via `{{< param variable_name >}}` in workshop markdown
- **Terminal** — available as `$VARIABLE_NAME` environment variable in the shell
- **Session config** — available as `$(variable_name)` in session-level workshop definition sections (`spec.session.*`)
- **Environment config** — available as `$(variable_name)` in environment-level workshop definition sections (`spec.environment.*`)

| Variable | Description | Instructions | Terminal | Session config | Environment config |
|----------|-------------|:---:|:---:|:---:|:---:|
| `assets_repository` | Host name of the workshop environment assets repository when enabled | Y | Y | Y | Y |
| `cluster_domain` | Internal domain used by the Kubernetes cluster (usually `cluster.local`) | Y | Y | Y | Y |
| `config_password` | Unique random password for accessing the workshop session configuration | Y | Y | Y | |
| `environment_name` | Name of the workshop environment | Y | Y | Y | Y |
| `image_repository` | Host name of the image repository associated with the cluster or training portal | Y | Y | Y | Y |
| `ingress_class` | Ingress class Educates is configured to use | Y | Y | Y | Y |
| `ingress_domain` | Domain for generated hostnames of ingress routes | Y | Y | Y | Y |
| `ingress_port` | Port number for workshop session ingress (usually 80 or 443, but may differ for Docker deployment) | Y | Y | Y | Y |
| `ingress_port_suffix` | Port number with colon prefix for ingress; empty when standard ports 80 or 443 | Y | Y | Y | Y |
| `ingress_protocol` | Protocol (`http` or `https`) used for ingress routes | Y | Y | Y | Y |
| `ingress_secret` | Name of the Kubernetes secret containing the wildcard TLS certificate for ingresses | | | Y | Y |
| `kubernetes_api_url` | URL for accessing the Kubernetes API (only valid from the workshop terminal) | Y | Y | | |
| `kubernetes_ca_crt` | Public certificate required when accessing the Kubernetes API URL | Y | | | |
| `kubernetes_token` | Kubernetes access token of the service account the session runs as | Y | | | |
| `oci_image_cache` | Hostname of the workshop environment OCI image cache when enabled | Y | Y | Y | Y |
| `pathway_name` | Name of the pathway for workshop instructions when in use | Y | | | |
| `platform_arch` | CPU architecture the workshop container runs on (`amd64` or `arm64`) | Y | Y | Y | Y |
| `policy_engine` | Name of the security policy engine applied to workshops (usually `kyverno`) | Y | Y | | |
| `policy_name` | Name of the security policy restricting workload types when session has Kubernetes access | Y | Y | | |
| `service_account` | Name of the service account in the workshop namespace that the session pod runs as | | | Y | Y |
| `services_password` | Unique random password for arbitrary services deployed with a workshop | Y | Y | Y | |
| `session_hostname` | Host name of the workshop session instance | Y | Y | Y | |
| `session_id` | Short identifier for the workshop session (unique within its workshop environment) | Y | Y | Y | |
| `session_name` | Name of the workshop session (unique within the Kubernetes cluster) | Y | Y | Y | |
| `session_namespace` | Namespace linked to the session for deploying applications (when Kubernetes access is enabled) | Y | Y | Y | |
| `session_url` | Full URL for accessing the workshop session dashboard | Y | Y | Y | |
| `ssh_keys_secret` | Name of the Kubernetes secret containing the SSH keys for the session | | | Y | |
| `ssh_private_key` | Private part of the unique SSH key pair generated for the session | Y | | Y | |
| `ssh_public_key` | Public part of the unique SSH key pair generated for the session | Y | | Y | |
| `storage_class` | Storage class Educates is configured to use | Y | Y | Y | Y |
| `training_portal` | Name of the training portal hosting the workshop | Y | Y | Y | Y |
| `workshop_description` | Description of the workshop from the workshop definition | Y | | | |
| `workshop_environment_uid` | Resource `uid` for the `WorkshopEnvironment` resource | | | Y | |
| `workshop_image` | Image used to deploy the workshop container | | | Y | Y |
| `workshop_image_pull_policy` | Image pull policy of the workshop container image | | | Y | Y |
| `workshop_name` | Name of the workshop | Y | Y | Y | Y |
| `workshop_namespace` | Name of the namespace used for the workshop environment | Y | Y | Y | Y |
| `workshop_session_uid` | Resource `uid` for the `WorkshopSession` resource | | | Y | |
| `workshop_title` | Title of the workshop (may be overridden when a specific pathway is selected) | Y | | | |
| `workshop_version` | Version tag from the workshop image (`latest` if not known) | | | Y | Y |

**Notes:**

- The terminal availability column reflects commonly observed environment variables. The exact set may vary depending on platform version and enabled applications.
- Variables only available in workshop instructions (such as `pathway_name`, `workshop_description`, and `workshop_title`) are specific to the content rendering context and are not exposed as environment variables or workshop definition data variables.

## Environment-Level Download Variables

A further subset of data variables is available specifically in configuration sections for downloads performed during workshop environment setup:

`assets_repository`, `cluster_domain`, `environment_name`, `image_repository`, `ingress_domain`, `ingress_port_suffix`, `ingress_protocol`, `oci_image_cache`, `platform_arch`, `training_portal`, `workshop_name`, `workshop_namespace`, `workshop_version`.

## Request Data Variables

Request data variables can be used in `spec.session.request.objects` sections of the workshop definition. These include all session data variables plus additional user information from the training portal:

| Variable | Description |
|----------|-------------|
| `username` | Name of the user as registered by the training portal |
| `first_name` | First name of the registered user |
| `last_name` | Last name of the registered user |
| `email` | Email address of the registered user |

Whether these contain useful values depends on the authentication method configured for the training portal. With anonymous authentication, `username` is a UUID and other fields are empty.

**IMPORTANT:** Request data variables and user information variables can only be used in `request.objects`. They cannot be used in `session.objects` or `environment.objects`.

## Application-Specific Data Variables

When certain session applications are enabled, additional data variables become available. These are conditional on the application being enabled in the workshop definition.

### Image Registry Variables

When the image registry is enabled (`spec.session.applications.registry.enabled: true`), the following additional variables are available as data variables in workshop instructions, as environment variables in the terminal, and as session data variables in the workshop definition:

| Variable | Terminal env var | Description |
|----------|-----------------|-------------|
| `registry_host` | `REGISTRY_HOST` | Host name for the session image registry |
| `registry_auth_file` | `REGISTRY_AUTH_FILE` | Location of the Docker configuration file (equivalent to `$HOME/.docker/config.json`) |
| `registry_username` | `REGISTRY_USERNAME` | Username for accessing the image registry |
| `registry_password` | `REGISTRY_PASSWORD` | Password for accessing the image registry (different per session) |
| `registry_auth_token` | `REGISTRY_AUTH_TOKEN` | Username and password separated by a colon and base64 encoded |
| `registry_secret` | `REGISTRY_SECRET` | Name of a Kubernetes secret of type `kubernetes.io/dockerconfigjson` in the session namespace containing registry credentials |

The registry URL uses the same HTTP protocol scheme as the workshop sessions, inherited from `ingress_protocol`.

Other session applications may provide additional data variables when enabled. These are documented in the Educates documentation sections for the specific application features.
