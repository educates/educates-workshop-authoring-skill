# Hugo Shortcodes Reference

Workshop instructions are rendered by Hugo and support shortcodes — special tags that insert dynamic content or styled elements into the page. This document covers the shortcodes available in Educates beyond the clickable actions documented elsewhere.

## Admonition Shortcodes

Admonitions are styled callout blocks that visually highlight important information. Use them to draw attention to notes, warnings, or critical information that the learner should not miss.

### Note

Renders as a blue callout box. Use for supplementary information, tips, or helpful context that is not essential to completing the step but adds understanding.

```markdown
{{</* note */>}}
The `SESSION_NAMESPACE` environment variable is also available in the terminal
if you prefer to use it instead of the Hugo parameter.
{{</* /note */>}}
```

### Warning

Renders as a yellow callout box. Use for cautions — information the learner should be aware of to avoid mistakes or unexpected behavior.

```markdown
{{</* warning */>}}
Deleting the ConfigMap will cause the application to revert to its default
configuration. Make sure you have saved any changes before proceeding.
{{</* /warning */>}}
```

### Danger

Renders as a red callout box. Use for critical warnings — actions that could cause data loss, break the workshop environment, or have other irreversible consequences.

```markdown
{{</* danger */>}}
Running this command will delete all resources in the namespace. This
action cannot be undone.
{{</* /danger */>}}
```

### When to Use Admonitions

- Use **note** for optional background information, alternative approaches, or "good to know" context
- Use **warning** for common pitfalls, prerequisites that must be met, or actions that may have unexpected side effects
- Use **danger** sparingly — only for truly destructive or irreversible operations
- Do not overuse admonitions. If every paragraph is a callout, none of them stand out. Reserve them for information that genuinely needs visual emphasis

## Data Variable Shortcode

The `param` shortcode inserts the value of a data variable into the rendered page:

```markdown
Deploy the application to the `{{</* param session_namespace */>}}` namespace.
```

This works in both prose and inside clickable action blocks. See [data-variables-reference.md](data-variables-reference.md) for the complete list of available data variables.

## Base URL Shortcode

The `baseurl` shortcode returns the base URL path for referencing static assets within the workshop content:

```markdown
![Architecture diagram]({{</* baseurl */>}}/images/architecture.png)
```

See [images-in-workshop-pages.md](images-in-workshop-pages.md) for detailed guidance on including images in workshop pages.

## Pathway Conditional Rendering

When a workshop uses pathways (multiple instruction sequences defined in `workshop/config.yaml`), the `pathway` shortcode conditionally includes content based on the active pathway:

```markdown
{{</* pathway python */>}}
This section only appears when the Python pathway is selected.
{{</* /pathway */>}}

{{</* pathway java */>}}
This section only appears when the Java pathway is selected.
{{</* /pathway */>}}
```

This is useful for workshops that teach the same concepts in multiple languages or frameworks while sharing common instruction pages.

## Custom Shortcodes

Authors can create custom Hugo shortcodes by placing template files in the `workshop/layouts/shortcodes/` directory. Each file defines a shortcode with the same name as the file (without the `.html` extension).

For example, creating `workshop/layouts/shortcodes/architecture.html` would make `{{</* architecture */>}}` available in workshop instructions.

Custom shortcodes follow standard Hugo shortcode conventions. This is an advanced feature — most workshops only need the built-in shortcodes documented above.

## Shortcode Syntax Notes

Hugo shortcodes use `{{</* */>}}` delimiters (angle brackets). Do not confuse them with Go template syntax `{{ }}` (double curly braces) which is not available in workshop instructions.

The `/*` and `*/` shown in documentation examples are Hugo's shortcode escaping mechanism — they prevent the shortcode from being rendered when displaying the syntax itself. In actual workshop instructions, omit them:

```
Documentation example:  {{</* note */>}}
Actual usage:           {{< note >}}
```
