---
description: "Update runebook entries after code changes. Usage: /runebook-update [type/name]"
allowed-tools: Read, Edit, Write, Bash, Glob, Grep, LS, Task, AskUserQuestion
---

# Runebook Update

Update runebook entries: `{{ARGS}}`

Invoke the runebook:runebook-implementation skill, action: **update**

The entry name is: `{{ARGS}}`

If a name was provided, update that specific entry by re-reading its source files. If no name was provided, auto-detect changes from recent git history and update all affected entries.
