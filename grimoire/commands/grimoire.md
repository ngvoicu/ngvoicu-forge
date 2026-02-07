---
description: "Research suite — auto-dispatches to specialized agents based on your query. Usage: /grimoire <query>"
allowed-tools: Read, Edit, Write, Bash, Glob, Grep, LS, Task, WebSearch, WebFetch, AskUserQuestion
---

# Grimoire

Run research using the grimoire agent suite.

Invoke the grimoire:grimoire-research skill.

Arguments: `{{ARGS}}`

**Default behavior: auto-dispatch.** Grimoire analyzes your query and picks the right agents automatically — codebase analysis, library docs, best practices, git history, DB schema, or API contracts. You don't need to specify which stream to use.

If the first word of `{{ARGS}}` matches a known stream name, use that single stream for a quick focused lookup:

| First word | Action | Stream |
|------------|--------|--------|
| `docs` | lookup | grimoire-docs |
| `practices` | lookup | grimoire-practices |
| `codebase` | lookup | grimoire-codebase |
| `history` | lookup | grimoire-history |
| `schema` | lookup | grimoire-schema |
| `contracts` | lookup | grimoire-contracts |
| *(empty)* | streams | show available streams |
| *(anything else)* | **research** | **auto-dispatch to relevant agents** |

Pass the parsed action and remaining arguments to the skill.
