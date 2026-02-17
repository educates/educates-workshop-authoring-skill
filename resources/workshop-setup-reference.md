# Workshop Environment Setup Reference

Workshop files can include scripts and configuration to customize the container environment before and during the workshop session. This covers installing additional tools, setting environment variables, running background services, and customizing terminal sessions.

All paths below are relative to the workshop root directory. These files are shipped as part of the workshop content and are processed automatically by the Educates platform when the workshop container starts.

## Directory Structure Overview

```
workshop/
├── setup.d/                    # Scripts run on container start
│   └── *.sh
├── profile.d/                  # Scripts run inline during container init
│   └── *.sh
├── profile                     # Sourced for each terminal session
├── supervisor/                 # Background application configs
│   └── *.conf
├── terminal/                   # Per-session terminal command overrides
│   ├── 1.sh
│   ├── 2.sh
│   └── 3.sh
└── terminal.sh                 # Default terminal command override
```

## Setup Scripts (`workshop/setup.d`)

To run a script when the container starts — for example, to install tools, pre-create resource files, or perform one-time initialization — add an executable shell script to the `workshop/setup.d` directory. The script must have a `.sh` suffix to be recognised and run.

Scripts are executed in alphabetical order by filename. Use numeric prefixes to control ordering when one script depends on another (e.g., `01-install-tools.sh`, `02-setup-data.sh`).

Setup scripts run with the workshop user home directory as the current working directory.

### Idempotency Requirement

If the container is restarted, setup scripts run again in the new container. Scripts must be tolerant of being run more than once. This is especially important for scripts that interact with the Kubernetes REST API via `kubectl` or other tools — ensure that create operations use `apply` or check for existing resources before creating them.

### Templating Files with `envsubst`

A common use of setup scripts is to fill out values in resource files using environment variables. The `envsubst` utility replaces variable references in an input file with their current values:

```shell
#!/bin/bash

envsubst < frontend/ingress.yaml.in > frontend/ingress.yaml
```

A reference of the form `${INGRESS_DOMAIN}` in the input file is replaced with the value of the `INGRESS_DOMAIN` environment variable. Refer to [data-variables-reference.md](data-variables-reference.md) for the environment variables available in the workshop container.

### Setting Persistent Environment Variables

If a setup script needs to set environment variables that should be available in subsequent scripts, the interactive terminal, and other processes in the container, write the variables to the file specified by the `WORKSHOP_ENV` environment variable:

```shell
#!/bin/bash

echo "MY_VARIABLE=some-value" >> $WORKSHOP_ENV
```

Write variables in `.env` file format (`NAME=VALUE`, one per line). These variables become available across the entire workshop session.

### Installing Additional Tools

Setup scripts can download and install additional command-line tools. The workshop container PATH includes `$HOME/.local/bin` and `$HOME/bin` — place executables in either location and they are found automatically without further configuration. See [workshop-tools-reference.md](workshop-tools-reference.md) for the PATH defaults and pre-installed tools.

The workshop user is a normal (non-root) user — there is no ability to use `sudo` to install system packages. Tools must be installed as standalone binaries or archives downloaded into user-writable locations such as `$HOME/.local/bin`. The base image OS is Fedora, which may be relevant when downloading platform-specific binaries.

For applications with many files, the workshop user can create directories under `/opt` rather than placing everything in `$HOME/.local/bin`. Add a custom `bin` directory to the PATH by writing it to `$WORKSHOP_ENV` from the setup script so it is available to subsequent setup scripts and the rest of the session. If the PATH addition is only needed for terminal sessions (not during setup), it can alternatively be added in `workshop/profile`.

Example — installing a single-binary tool:

```shell
#!/bin/bash

mkdir -p $HOME/.local/bin

curl -sL https://example.com/tool-v1.0.tar.gz | tar xz -C /tmp
mv /tmp/tool $HOME/.local/bin/tool
chmod +x $HOME/.local/bin/tool
```

Example — installing an application with multiple files under `/opt`:

```shell
#!/bin/bash

mkdir -p /opt/myapp
curl -sL https://example.com/myapp-v2.0.tar.gz | tar xz -C /opt/myapp

echo "PATH=/opt/myapp/bin:\$PATH" >> $WORKSHOP_ENV
```

### File Permissions with Custom Images

If the workshop uses a custom workshop image and the setup script creates or updates files in the file system, ensure the workshop image is built with correct file permissions to allow those updates.

## Profile Scripts (`workshop/profile.d`)

Scripts with a `.sh` extension in the `workshop/profile.d` directory are executed once, inline with the scripts used to initialize the overall container environment. This happens after the workshop content has been downloaded and after `setup.d` scripts have been run.

Because `profile.d` scripts are executed inline, any environment variables that are set and exported from these scripts flow through and are available in:

- Rendered workshop instructions (Hugo data variables)
- Terminal sessions
- Other container processes

```shell
#!/bin/bash

export CUSTOM_ENDPOINT="https://api.example.com/v1"
```

### Deprecation Warning

The `profile.d` script feature is likely to be deprecated and removed in a future version. Use the `$WORKSHOP_ENV` mechanism from a `setup.d` script instead — it provides the same capability of making environment variables available across the workshop session.

## Shell Profile (`workshop/profile`)

When the file `workshop/profile` exists, it is sourced automatically when each terminal shell session is created. Use this file only for customizing the interactive shell environment:

- Modifying the terminal prompt
- Setting up command line completion
- Defining shell aliases or functions

```shell
# Customize the terminal prompt — show working directory in a distinct color
export PS1="\[\033[1;31m\][\w] $ \[\033[0m\]"

# Alternative — add a blank line before each prompt for visual separation
# between command outputs, making it easier to find where each command starts
export PS1="\n\[\033[33m\][\w] $ \[\033[0m\]"

# Enable completion for a custom tool installed by a setup script
# (completion for standard tools like kubectl is already configured automatically)
source <(mytool completion bash)
```

**Important considerations:**

- Do not replace `$HOME/.bash_profile` — it contains default setup required by the workshop environment. The `workshop/profile` mechanism is the correct way to add shell customizations.
- The `workshop/profile` script is invoked separately for every terminal session. Keep it lightweight — avoid actions that query APIs or perform expensive operations.
- Environment variables set in `workshop/profile` are **not** available when rendering workshop instructions. To set environment variables that need to be available in instructions, use a `setup.d` script with `$WORKSHOP_ENV` or a `profile.d` script instead.

## Background Applications (`workshop/supervisor`)

To run a background application for the life of the workshop session — such as a web server, database, or custom service — integrate it with the supervisor daemon that runs inside the container. Add a configuration file with a `.conf` extension to the `workshop/supervisor` directory.

The configuration file snippet follows the supervisor daemon format:

```text
[program:myapplication]
process_name=myapplication
command=/opt/myapplication/sbin/start-myapplication
stdout_logfile=/proc/1/fd/1
stdout_logfile_maxbytes=0
redirect_stderr=true
```

The application should send logging output to `stdout` or `stderr`. The configuration snippet should direct log output to `/proc/1/fd/1` so it is captured in the container log file.

### Why Use Supervisor Instead of Setup Scripts

Although a `setup.d` script could start a background process, if that process stops it will not be restarted. The supervisor daemon monitors the application and restarts it automatically if it exits.

### Runtime Control

To restart or stop the managed application from within the workshop terminal, use the `supervisorctl` command:

```shell
supervisorctl status myapplication
supervisorctl restart myapplication
supervisorctl stop myapplication
```

## Terminal Command Overrides (`workshop/terminal`)

By default, each terminal session starts an interactive `bash` shell. To override this and run a specific command instead, provide an executable shell script in the `workshop/terminal` directory.

### Per-Session Overrides

The script file name must match the terminal session name: `workshop/terminal/<session>.sh`. The default terminal session names are `1`, `2`, and `3`.

Example — running `k9s` in terminal session 1:

```shell
#!/bin/bash

exec k9s
```

If the command exits, the terminal session is marked as exited and must be reloaded. To auto-restart the command, wrap it in a loop:

```shell
#!/bin/bash

while true; do
    k9s
    sleep 1
done
```

To display a banner before starting an interactive shell:

```shell
#!/bin/bash

echo
echo "Your session namespace is $SESSION_NAMESPACE."
echo

exec bash
```

### Default Override for All Sessions

To provide a default command for all terminal sessions regardless of name, add an executable shell script at `workshop/terminal.sh` (in the `workshop` directory, not inside `workshop/terminal/`).

A per-session script (`workshop/terminal/<session>.sh`) takes precedence over the default script (`workshop/terminal.sh`) when both exist.

## See Also

- [Workshop Tools Reference](workshop-tools-reference.md) — Pre-installed command-line tools and PATH configuration
- [Data Variables Reference](data-variables-reference.md) — Environment variables and data variables available in the workshop container
