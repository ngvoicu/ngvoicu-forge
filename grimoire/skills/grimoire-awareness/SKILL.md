---
name: grimoire-awareness
description: "Proactive research awareness. Triggers on: 'look up docs for', 'how do I use X library', 'what's the best way to', 'research this', 'find best practices for', 'check the docs', or when adding new dependencies, using unfamiliar library APIs, or implementing security-sensitive features. Suggests or auto-invokes grimoire research when guessing would be dangerous."
user-invocable: false
---

# Grimoire Awareness

## Trigger Phrases

Activate this skill when the user says anything matching these patterns. **Default: use `/grimoire <query>` for auto-dispatch.** Grimoire picks the right agents based on the query. Only use a specific stream prefix when the user explicitly asks for a single stream.

**Research and understanding requests (auto-dispatch):**
- "research this", "research X", "investigate X"
- "I need to understand X before building"
- "what's the best way to X", "best practices for X"
- "how should I implement X", "is this the right approach for X"
- "how does our X work", "where is X implemented"
- "why is X like this", "has anyone tried X before"
- When any of these are detected, invoke `/grimoire <query>` — let auto-dispatch pick the agents

**Explicit single-stream requests (use the stream prefix):**
- "look up docs for X", "check the docs for X" → `/grimoire docs <X>`
- "check git history for X", "who changed X" → `/grimoire history <X>`
- "what's the schema for X", "check the migrations" → `/grimoire schema <X>`
- "check the X API docs", "what are the rate limits" → `/grimoire contracts <X>`

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
- Suggest: "This is security-sensitive — want me to research first? `/grimoire <topic>`"

### Before major refactors (suggest)

If the user is about to restructure significant portions of the codebase:
- Suggest: "Before refactoring, want me to research this area? `/grimoire <area>`"

### When debugging mysterious behavior (suggest)

If the user is confused about why something works a certain way:
- Suggest: "Want me to research why this works this way? `/grimoire <area>`"

## Integration with Other Plugins

**With specsmith:** When specsmith Phase 2 runs, grimoire agents replace the inline subagent definitions — providing richer, more focused research through dedicated agents.

**With runebook:** Grimoire's codebase analysis complements runebook's living documentation. Use runebook entries as input context for grimoire research, and update runebook entries based on grimoire findings.

