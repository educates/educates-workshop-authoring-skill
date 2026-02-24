# Workshop Design Principles

This document describes the design philosophy for Educates workshops. Follow these principles by default when creating workshops. If the person requesting the workshop explicitly asks for a different experience (e.g., learners typing code themselves), adjust accordingly — but treat guided instruction as the standard approach.

## Guided Experience Philosophy

Educates workshops provide a guided experience where all code interaction is driven through clickable actions embedded in the workshop instructions. The goal is that learners can follow along, observe behavior, and build understanding without needing to context-switch into typing code or commands themselves. All interaction should feel like clicking through a well-paced guided demo, with the workshop instructions orchestrating the editor and terminal on the learner's behalf.

"Hands-on" does not mean learners type code from scratch. It means learners actively engage with code — viewing it, running it, and seeing it change — through clickable actions that keep them focused on the concepts rather than on mechanics.

## The No Manual Typing Rule

Workshop instructions must not ask learners to type commands into the terminal or write code into the editor by hand. Every interaction should be handled through the appropriate clickable action:

- **Never** write instructions like "run the following command" and show a plain code block the learner must copy-paste or type. Always provide a `terminal:execute` clickable action.
- **Never** write instructions like "add the following lines to the file" and expect the learner to type them. Always use an editor clickable action (`editor:create-file`, `editor:append-lines-to-file`, `editor:replace-matching-text`, etc.).
- **Never** direct learners to use a text editor like `nano` or `vim` in the terminal. Use editor clickable actions to make file changes.

The `workshop:copy` and `workshop:copy-and-edit` actions copy text to the clipboard. These are acceptable as a fallback when content must be customized per-learner (e.g., a username or personal token) and cannot be pre-scripted using data variables. They should not be the default mechanism for code or commands that could be handled by `terminal:execute` or editor actions.

## How Learners Interact with Code

Workshop instructions orchestrate three kinds of code interaction: viewing, running, and modifying. Each uses a specific set of clickable actions.

### Viewing Code

Pre-written files are provided in the workshop's `exercises/` directory (or created inline via `editor:create-file` at the appropriate point in the instructions). Learners view code using:

- **`editor:open-file`** — opens a file in the embedded VS Code editor so the learner can see the full contents
- **`editor:select-matching-text`** — highlights a specific section of an open file to draw attention to key concepts, patterns, or the code that is about to be discussed
- **`editor:select-lines-in-range`** — highlights a range of lines when the selection is best described by line numbers rather than a text match

Use these actions to guide the learner's eye to the relevant parts of the code before explaining what it does.

### Running Code

Learners execute code and commands via `terminal:execute` clickable actions in the workshop instructions. They do not type commands into the terminal themselves.

When a command produces output that the instructions will discuss, tell the learner what to expect before or after the action so they know what to look for. When the output appears in a dashboard tab other than the terminal (e.g., a web application accessed through the session proxy), include a `dashboard:open-dashboard` or `dashboard:reload-dashboard` action to switch the learner to the correct tab. See [workshop-dashboard-reference.md](workshop-dashboard-reference.md) for detailed patterns on managing tab visibility.

### Modifying Code

Where a workshop calls for code changes (e.g., adding a decorator, modifying a configuration value, refactoring a function), these are performed through editor clickable actions — not by asking the learner to edit files manually. Choose the action that best fits the change:

| Change Type | Recommended Action |
|---|---|
| Replace a short or single-line piece of text | `editor:replace-matching-text` |
| Replace a multi-line block of text | `editor:select-matching-text` then `editor:replace-text-selection` |
| Add lines after a specific location | `editor:append-lines-after-match` |
| Append lines to end of file | `editor:append-lines-to-file` |
| Insert lines at a specific position | `editor:insert-lines-before-line` or `editor:insert-lines-after-line` |
| Delete specific lines | `editor:delete-matching-lines` or `editor:delete-lines-in-range` |
| Replace a range of lines | `editor:replace-lines-in-range` |
| Create or overwrite a file entirely | `editor:create-file` |
| Modify YAML structure | `editor:set-yaml-value`, `editor:merge-yaml-values`, etc. |

When making incremental changes, explain what is being changed and why before the clickable action so the learner understands the purpose of the modification. For multi-line replacements, prefer the two-step approach: use `editor:select-matching-text` to highlight the target block first, then provide explanatory commentary so the learner can read the existing code and understand what will change, and finally use `editor:replace-text-selection` to apply the replacement. This avoids the problem of large blocks being replaced instantly before the learner can see what was there. For short or single-line replacements, `editor:replace-matching-text` is fine. After any modification, consider using `editor:select-matching-text` to highlight the changed code so the learner can see exactly what was modified.

When a workshop progresses through a series of code examples and the next example would replace all or most of a file's contents, avoid using replacement actions to overwrite the file. Instead, provide the next example as a separate pre-created file in the `exercises/` directory and open it with `editor:open-file`. This lets the learner compare both versions. This guidance applies to illustrative example code introduced for teaching — it does not apply when modifying application files that genuinely need to be updated in place as part of the exercise workflow.

## The Role of the exercises/ Directory

The `exercises/` directory serves as a pre-populated workspace, not a blank slate. Workshop authors should provide all the code the learner needs as starter files so the learner's job is to observe, understand, and make guided incremental changes — not to author code from scratch.

### What to Pre-populate

Place files in `exercises/` before the session starts when the code:

- Is a starting point the learner will build on through the workshop
- Is needed early in the workshop (e.g., a project skeleton, configuration files, sample data)
- Would be tedious or error-prone to create via clickable actions
- Represents the "given" state for the exercise — code that exists so the learner can focus on the new concepts being introduced

### What to Create Inline

Use `editor:create-file` during the workshop when:

- The file is introduced at a specific point in the narrative and creating it is part of the lesson flow
- The learner needs to see the file appear as a result of an action they take
- The file content depends on earlier steps in the workshop

### Starter Code vs. Incremental Changes

A common pattern is to provide a working baseline in `exercises/` and then use editor clickable actions during the workshop to make incremental modifications. This keeps each step focused on a single concept:

1. Learner views the starter code via `editor:open-file`
2. Instructions explain the relevant section, using `editor:select-matching-text` to highlight it
3. A clickable action applies the change — for short replacements, `editor:replace-matching-text` works well; for multi-line replacements, use `editor:select-matching-text` to highlight the target block, explain what will change, then `editor:replace-text-selection` to apply it
4. The learner runs the modified code via `terminal:execute` to see the effect
5. Instructions discuss what changed and why

This pattern gives learners a clear before-and-after for each concept without requiring them to build anything from scratch.
