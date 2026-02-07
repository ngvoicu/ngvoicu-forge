---
name: chisel-awareness
description: "Proactive awareness for surgical code edits. Triggers on: 'rename X to Y', 'move X to Y', 'replace X with Y', 'change all X to Y', 'update the color', 'fix the import', 'change the font', or when renaming/moving files manually without updating references. Auto-invokes chisel for explicit rename/move/replace requests. Suggests chisel for ambiguous edits."
user-invocable: false
---

# Chisel Awareness

## Trigger Phrases

Activate this skill when the user says anything matching these patterns:

**Rename requests:**
- "rename X to Y", "rename the file X", "rename the component X"
- "change the name of X to Y", "call it Y instead of X"
- "refactor X to Y" (when the user means rename, not restructure)
- When any of these are detected, invoke `/chisel rename <details>`

**Move requests:**
- "move X to Y", "move this file to", "put X in the Y folder"
- "relocate X", "reorganize the files"
- When any of these are detected, invoke `/chisel move <details>`

**Replace requests:**
- "replace X with Y", "change all X to Y", "find and replace X"
- "swap X for Y", "update all occurrences of X"
- "change X from A to B across the project"
- When any of these are detected, invoke `/chisel replace <details>`

**Small edit requests:**
- "change the color", "update the font", "fix the padding"
- "tweak the margin", "adjust the spacing", "update the config"
- "change the value of X to Y"
- When any of these are detected, invoke `/chisel edit <details>`

## Proactive Triggers

Chisel uses two escalation levels:
- **Auto-invoke** — run without asking for explicit rename/move/replace requests
- **Suggest** — inform the user and offer to run for ambiguous or risky operations

### Explicit operations (auto-invoke)

When the user explicitly requests a rename, move, or replace operation:
- Invoke `/chisel` directly — these operations are unambiguous

### Manual file operations observed (auto-invoke)

If you observe in the conversation that a file was renamed or moved without chisel (e.g., user ran a raw `mv` command, or you notice broken imports after a file move):
- Invoke `/chisel` to check for and fix broken references
- Say: "File was moved without chisel — fixing references."

### Ambiguous scope (suggest)

If a replacement would match a very common word or the scope is unclear:
- Suggest: "This could affect many files. Want me to run `/chisel replace <details>` with a preview first?"

### Large refactoring patterns (suggest)

If the user is renaming a concept across the codebase (e.g., changing a domain term):
- Suggest: "This is a large rename. Want me to use `/chisel rename <details>` to handle all references?"

## Integration with Other Plugins

**With specsmith:** When implementing a spec that requires file renames or moves, use chisel to ensure all references are updated. After chisel completes, mark the spec step as done.

**With runebook:** After chisel rename/move operations, update runebook entries that reference the affected files. Chisel changes `source_files` paths in runebook frontmatter.

**With grimoire:** Before large renames affecting many files, use `/grimoire codebase <area>` to understand the full scope of what will be affected.

**With uispec:** When renaming endpoints or API handlers, use chisel to update both the source files and the corresponding uispec endpoint spec files.
