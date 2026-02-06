---
description: "Research suite — invoke specialized agents for docs, best practices, codebase, history, schema, or API contracts. Usage: /grimoire <stream> <query> or /grimoire research <context>"
allowed-tools: Read, Edit, Write, Bash, Glob, Grep, LS, Task, WebSearch, WebFetch, AskUserQuestion
---

# Grimoire

Run research using the grimoire agent suite.

Invoke the grimoire:grimoire-research skill.

Arguments: `{{ARGS}}`

Parse the first word of `{{ARGS}}` as the stream name:

| First word | Action | Stream |
|------------|--------|--------|
| `docs` | lookup | grimoire-docs |
| `practices` | lookup | grimoire-practices |
| `codebase` | lookup | grimoire-codebase |
| `history` | lookup | grimoire-history |
| `schema` | lookup | grimoire-schema |
| `contracts` | lookup | grimoire-contracts |
| `research` | research | all relevant agents |
| *(empty)* | streams | show available streams |
| *(other)* | research | treat entire input as context |

Pass the parsed action and remaining arguments to the skill.
