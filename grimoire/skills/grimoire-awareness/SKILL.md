---
name: grimoire-awareness
description: "Proactive research awareness. Triggers on: 'look up docs for', 'how do I use X library', 'what's the best way to', 'research this', 'find best practices for', 'check the docs', or when adding new dependencies, using unfamiliar library APIs, or implementing security-sensitive features. Suggests or auto-invokes grimoire research when guessing would be dangerous."
---

# Grimoire Awareness

## Trigger Phrases

Activate this skill when the user says anything matching these patterns:

**Documentation requests:**
- "look up docs for X", "how do I use X", "what's the API for X"
- "check the docs for X", "find the docs for X"
- "what version of X", "is X compatible with Y"
- When any of these are detected, invoke `/grimoire docs <X>`

**Best practices requests:**
- "what's the best way to X", "best practices for X"
- "how should I implement X", "is this the right approach for X"
- "find best practices for X", "common mistakes with X"
- When any of these are detected, invoke `/grimoire practices <X>`

**Codebase analysis requests:**
- "how does our X work", "where is X implemented"
- "what pattern do we use for X", "find similar code to X"
- "what files would I need to change for X"
- When any of these are detected, invoke `/grimoire codebase <X>`

**History requests:**
- "why is X like this", "who changed X", "when was X added"
- "has anyone tried X before", "what happened with X"
- "check git history for X"
- When any of these are detected, invoke `/grimoire history <X>`

**Schema requests:**
- "what's the schema for X", "how is X stored in the database"
- "what tables are related to X", "check the migrations for X"
- When any of these are detected, invoke `/grimoire schema <X>`

**API contract requests:**
- "what does the X API look like", "check the X API docs"
- "what are the rate limits for X", "how does X auth work"
- When any of these are detected, invoke `/grimoire contracts <X>`

**Full research requests:**
- "research this", "research X", "investigate X"
- "I need to understand X before building"
- When any of these are detected, invoke `/grimoire research <X>`

## Proactive Triggers

Grimoire uses two escalation levels:
- **Auto-invoke** — run without asking. Use when guessing would produce incorrect code.
- **Suggest** — inform the user and offer to run. Use for all other proactive triggers.

### When the user is guessing at library APIs (auto-invoke)

If you notice the user (or yourself) is uncertain about a library's API:
- Stop and invoke `/grimoire docs <library> <specific question>` instead of guessing

### When new dependencies are added (suggest)

If a `package.json`, `requirements.txt`, `Cargo.toml`, `go.mod`, or similar file is being modified to add a new dependency:
- Suggest: "New dependency detected. Want me to look up the docs? `/grimoire docs <package>`"

### When implementing security-sensitive features (suggest)

If the implementation involves authentication, authorization, cryptography, payment processing, PII handling, or input validation:
- Suggest: "This is security-sensitive — want me to research best practices first? `/grimoire practices <topic>`"

### Before major refactors (suggest)

If the user is about to restructure significant portions of the codebase:
- Suggest: "Before refactoring, want me to analyze the current patterns? `/grimoire codebase <area>`"

### When debugging mysterious behavior (suggest)

If the user is confused about why something works a certain way:
- Suggest: "Want me to check the git history to understand why? `/grimoire history <area>`"

## Integration with Other Plugins

**With specsmith:** When specsmith Phase 2 runs, grimoire agents replace the inline subagent definitions — providing richer, more focused research through dedicated agents.

**With runebook:** Grimoire's codebase analysis complements runebook's living documentation. Use runebook entries as input context for grimoire research, and update runebook entries based on grimoire findings.

