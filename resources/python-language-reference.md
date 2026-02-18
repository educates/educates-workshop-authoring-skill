# Python Language Reference

This guide covers what is needed when creating workshops that involve Python applications.

## Workshop Image

The default `base-environment` image includes Python 3.13 and is suitable for most Python workshops. There is no need to specify a custom image unless the workshop has specific requirements beyond what the base environment provides.

For workshops that require the Anaconda Python distribution — for example, those involving scientific computing packages, conda package management, or Jupyter notebook environments that rely on conda — use the `conda-environment` image instead:

```yaml
spec:
  workshop:
    image: conda-environment:*
    files:
    - image:
        url: "$(image_repository)/{workshop-name}-files:$(workshop_version)"
```

See the [Workshop Image Reference](workshop-image-reference.md) for more details on image selection and configuration.

## Package Management

### Virtual environment required

To install additional Python packages in the workshop environment, **you must use a Python virtual environment**. The system Python does not allow direct package installation — `pip` is limited to the per-user packages directory, and `uv pip install` requires an active virtual environment. Workshop instructions that attempt to install packages without a virtual environment will fail.

There are two ways to set up a virtual environment:

1. **Create a virtual environment explicitly** — suitable when working with standalone scripts or an existing `requirements.txt`.
2. **Use a uv project** — when the workshop scaffolds a project with `uv init`, `uv` manages the virtual environment automatically.

Both approaches are covered below.

### Prefer uv over pip

The workshop environment includes both `uv` and `pip`, but **use `uv` in workshop instructions rather than `pip`**. `uv` is significantly faster at resolving and installing packages, which keeps workshop exercises flowing smoothly.

### Creating a virtual environment explicitly

Create and activate a virtual environment, then install packages into it:

````markdown
```terminal:execute
command: uv venv .venv && source .venv/bin/activate
```
````

````markdown
```terminal:execute
command: uv pip install flask requests
```
````

Or install from a requirements file:

````markdown
```terminal:execute
command: uv pip install -r requirements.txt
```
````

### Using a uv project

Use `uv init` to scaffold a new Python project:

````markdown
```terminal:execute
command: uv init myproject
```
````

This creates a project directory with `pyproject.toml`, `README.md`, and a `main.py` entry point. Some older documentation and tutorials refer to `hello.py` as the default entry point — that was the convention in earlier versions of `uv`. The version installed in the workshop environment generates `main.py`.

When working inside a uv project, use `uv add` to install dependencies. This records the dependency in `pyproject.toml` and installs it into a project-level virtual environment that `uv` creates and manages automatically:

````markdown
```terminal:execute
command: uv add flask requests
```
````

Run scripts within the project's virtual environment using `uv run`:

````markdown
```terminal:execute
command: uv run main.py
```
````

## Installing a Specific Python Version

The base image ships with Python 3.13. If the workshop requires a different Python version, use `uv python install` from a setup script to install it before the workshop session begins.

Create a setup script at `workshop/setup.d/install-python.sh`:

```shell
#!/bin/bash

uv python install 3.12
```

This downloads and installs the requested Python version. The `uv` tool may also automatically download a Python version if a project's configuration (such as `pyproject.toml`) requests one, so an explicit install step is not always necessary.

See the [Workshop Setup Reference](workshop-setup-reference.md) for details on setup scripts.

## Exercise Files

For Python workshops, the `exercises/` directory typically contains the starter project or application source code that users work with during the workshop. Common layouts include:

**Single Python project:**

```
exercises/
├── pyproject.toml
├── requirements.txt
└── src/
    └── myapp/
        ├── __init__.py
        └── main.py
```

**Simple script-based workshop:**

```
exercises/
├── app.py
└── requirements.txt
```

**Multiple modules or projects:**

```
exercises/
├── api/
│   ├── pyproject.toml
│   └── src/...
└── client/
    ├── pyproject.toml
    └── src/...
```

Place all source files under `exercises/` so terminals and the editor open to the correct location. See the exercises directory section in the main skill guide for details on why this matters.

## Running Python Applications

### Web applications (Flask, FastAPI, Django)

Python web frameworks typically listen on a specific port. Since the application runs inside the workshop container (started from the terminal), use a session ingress that proxies to the application port to make it accessible in the browser.

#### Workshop definition for a web application

Add session ingress and dashboard entries to expose the application and embed it as a dashboard tab. This example uses port 8080, but adjust the port to match the framework or application configuration:

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

#### Starting the application

A typical pattern is to start the application in one terminal and use the other terminal for additional commands:

**Flask:**

````markdown
```terminal:execute
command: flask run --port 8080
```
````

**FastAPI with uvicorn:**

````markdown
```terminal:execute
command: uvicorn main:app --host 0.0.0.0 --port 8080
```
````

Since these commands block the terminal, the split terminal layout is essential so the user has a second terminal available. The terminal application should already be configured with `layout: split` as per the standard workshop setup.

Once the application is running, reveal the dashboard tab so the user can interact with it:

````markdown
```dashboard:open-dashboard
name: App
```
````

### Command-line scripts

For workshops focused on scripting rather than web applications, run Python scripts directly:

````markdown
```terminal:execute
command: python app.py
```
````

## Anaconda / Conda Environments

When using the `conda-environment` image, the workshop has access to the `conda` package manager. This is useful for workshops involving data science, machine learning, or scientific computing where packages from conda channels are preferred.

Create a conda environment from an environment file:

````markdown
```terminal:execute
command: conda env create -f environment.yml
```
````

Activate the environment:

````markdown
```terminal:execute
command: conda activate myenv
```
````

Place the `environment.yml` file in the `exercises/` directory alongside any other project files.
