# Java Language Reference

This guide covers what is needed when creating workshops that involve Java applications.

## Workshop Image

The JDK workshop images provide a complete Java development environment. Each image includes:

- **JDK** — The full Java Development Kit for the corresponding version
- **Maven** — For building Maven-based projects
- **Gradle** — For building Gradle-based projects

Available JDK images:

| Image | Java version |
|---|---|
| `jdk8-environment:*` | Java 8 |
| `jdk11-environment:*` | Java 11 |
| `jdk17-environment:*` | Java 17 |
| `jdk21-environment:*` | Java 21 |

Choose the image that matches the Java version required by the application or framework being taught. When the workshop does not target a specific version, prefer `jdk21-environment:*` as the current long-term support release.

To use a JDK image, set `workshop.image` in the workshop definition:

```yaml
spec:
  workshop:
    image: jdk17-environment:*
    files:
    - image:
        url: "$(image_repository)/{workshop-name}-files:$(workshop_version)"
```

See the [Workshop Image Reference](workshop-image-reference.md) for more details on image selection and configuration.

## Exercise Files

For Java workshops, the `exercises/` directory typically contains the starter project or application source code that users work with during the workshop. Common layouts include:

**Single Maven project:**

```
exercises/
└── pom.xml
    src/
    ├── main/java/...
    └── test/java/...
```

**Single Gradle project:**

```
exercises/
└── build.gradle
    src/
    ├── main/java/...
    └── test/java/...
```

**Multiple modules or projects:**

```
exercises/
├── app/
│   ├── pom.xml
│   └── src/...
└── service/
    ├── pom.xml
    └── src/...
```

Place all source files under `exercises/` so terminals and the editor open to the correct location. See the exercises directory section in the main skill guide for details on why this matters.

## Build Commands

Maven and Gradle builds can produce large amounts of output. When running builds in workshop instructions, consider suppressing routine output to keep the terminal readable:

**Maven — quiet mode:**

````markdown
```terminal:execute
command: mvn -q package
```
````

**Gradle — quiet mode:**

````markdown
```terminal:execute
command: gradle -q build
```
````

Use the quiet flags for builds where the output is not instructionally relevant. Omit them when the build output itself is part of what the user needs to see (e.g., demonstrating compilation errors or test results).

## Spring Boot Applications

Many Java workshops involve Spring Boot. Spring Boot applications listen on port `8080` by default. Since the application runs inside the workshop container (started from the terminal), use a session ingress that proxies to `localhost:8080` to make it accessible in the browser.

### Workshop Definition for Spring Boot

Add session ingress and dashboard entries to the workshop definition to expose the application and embed it as a dashboard tab:

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

Because the application runs in the workshop container itself rather than as a Kubernetes Service, the `host` field is omitted — it defaults to `localhost`. See the [Workshop YAML Reference](workshop-yaml-reference.md) for the general session ingress and dashboard configuration.

### Running the Application

A typical pattern is to start the application in one terminal and use the other terminal for additional commands:

````markdown
```terminal:execute
command: mvn spring-boot:run
```
````

Since `mvn spring-boot:run` blocks the terminal, the split terminal layout is essential so the user has a second terminal available. The terminal application should already be configured with `layout: split` as per the standard workshop setup.

Once the application is running, reveal the dashboard tab so the user can interact with it:

````markdown
```dashboard:open-dashboard
name: App
```
````
