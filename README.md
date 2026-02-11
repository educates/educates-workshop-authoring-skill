# Educates Authoring Skill

A Claude Code skill for creating and configuring workshops for the [Educates](https://educates.dev/) interactive training platform.

## Features

- Create new workshop projects with proper directory structure
- Generate `workshop.yaml` configuration with appropriate session applications
- Author workshop instruction pages following Educates conventions

## Installation

### From GitHub Release

Pre-packaged `.skill` files are available from the [GitHub releases](https://github.com/educates/educates-authoring-skill/releases) page.

To download the latest release:

```bash
curl -fLO https://github.com/educates/educates-authoring-skill/releases/latest/download/educates-authoring-skill.skill
```

To download a specific version (replace `2.0` with the desired version tag):

```bash
curl -fLO https://github.com/educates/educates-authoring-skill/releases/download/2.0/educates-authoring-skill.skill
```

### From Source

1. Clone this repository:

   ```bash
   git clone https://github.com/educates/educates-authoring-skill.git
   cd educates-authoring-skill
   ```

2. Create an archive of the skill:

   ```bash
   zip -r educates-authoring-skill.skill . -x ".git/*" -x ".github/*"
   ```

### Importing into Claude

Once you have the `.skill` file, either downloaded from a GitHub release or built from source, install it using the Claude Code CLI:

```bash
claude skill install educates-authoring-skill.skill
```

Or in the Claude Code desktop app, go to **Settings > Skills** and use "Install from file" to select the `.skill` file.

### Usage

Once installed, invoke the skill with:

```
/educates-authoring
```

Or simply ask Claude to help create an Educates workshop and it will use the skill automatically based on context.

## Documentation

For more information about Educates, visit the official documentation at https://docs.educates.dev/
