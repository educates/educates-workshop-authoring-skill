# Educates Authoring Skill

A Claude Code skill for creating and configuring workshops for the [Educates](https://educates.dev/) interactive training platform.

## Features

- Create new workshop projects with proper directory structure
- Generate `workshop.yaml` configuration with appropriate session applications
- Author workshop instruction pages following Educates conventions

## Installation

### From Source

1. Clone this repository:

   ```bash
   git clone https://github.com/your-username/educates-authoring-skill.git
   cd educates-authoring-skill
   ```

2. Create a zip archive of the skill:

   ```bash
   zip -r educates-authoring.zip . -x ".git/*"
   ```

3. Import into Claude Code:

   - Open Claude Code settings
   - Navigate to Skills
   - Click "Install from file" and select the `educates-authoring.zip` file

### Usage

Once installed, invoke the skill with:

```
/educates-authoring
```

Or simply ask Claude to help create an Educates workshop and it will use the skill automatically based on context.

## Documentation

For more information about Educates, visit the official documentation at https://docs.educates.dev/
