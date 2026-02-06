---
description: "Fast surgical code editor — rename, move, replace, or make small edits with automatic reference updates. Usage: /chisel <action> <args>"
allowed-tools: Read, Edit, Write, Bash, Glob, Grep, LS, Task, AskUserQuestion
---

# Chisel

Perform a precise code edit using the chisel agent suite.

Invoke the chisel:chisel-ops skill.

Arguments: `{{ARGS}}`

Parse the first word of `{{ARGS}}` as the operation type:

| First word | Action | Description |
|------------|--------|-------------|
| `rename` | execute | Rename file/variable/class + update all references |
| `move` | execute | Move file/directory + update all import paths |
| `replace` | execute | Find-and-replace across files |
| `edit` | execute | Small targeted code change |
| *(empty)* | help | Show available operations with examples |
| *(other)* | execute | Auto-detect operation from natural language |

Pass the parsed action and remaining arguments to the skill.
