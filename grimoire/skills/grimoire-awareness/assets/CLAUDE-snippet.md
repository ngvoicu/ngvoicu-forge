## Grimoire (MANDATORY)

Research suite with 6 specialized agents. **ALWAYS research before guessing — never hallucinate library APIs or best practices.**

### When to Use Grimoire

| Situation | Command |
|-----------|---------|
| Using unfamiliar library API | `/grimoire docs <library> <question>` |
| Security-sensitive implementation | `/grimoire practices <topic>` |
| Before major changes or refactors | `/grimoire codebase <area>` |
| Understanding why code exists | `/grimoire history <area>` |
| Database/schema changes | `/grimoire schema <area>` |
| External API integration | `/grimoire contracts <service>` |
| Complex feature (full research) | `/grimoire <context>` (auto-dispatches all relevant agents) |

### Rules

- **NEVER guess library APIs** — always use `/grimoire docs` to look up exact signatures, parameters, and return types
- **ALWAYS research best practices** before implementing security-sensitive features (auth, crypto, payments, PII)
- **ALWAYS analyze the codebase** before major refactors to understand existing patterns and conventions
- **ALWAYS check API contracts** before integrating with external services to understand rate limits, auth, and error handling
- **Prefer `/grimoire <query>`** for complex features — it auto-dispatches all relevant agents in parallel

### Getting Started

Run `/grimoire` with no arguments to see all available research streams and examples.

### Commands

```
/grimoire                              # Show available streams
/grimoire docs <library> <query>       # Library documentation lookup
/grimoire practices <topic>            # Best practices & pitfalls
/grimoire codebase <area>              # Codebase analysis
/grimoire history <area>               # Git history & prior work
/grimoire schema <area>                # Database schema analysis
/grimoire contracts <service>          # External API contracts
/grimoire <context>                     # Auto-dispatch to relevant agents (full parallel research)
```

### Integration with Other Plugins

**With specsmith:** Grimoire automatically provides research agents for specsmith Phase 2. No manual invocation needed — specsmith detects grimoire and uses its agents for richer research.

**With runebook:** Read `.runebook/` entries as input context for grimoire research — they provide useful context about component behavior and dependencies.

**With chisel:** After grimoire research reveals files to modify, use `/chisel` for renames or moves to ensure all references stay intact.

**With uispec:** Use `/grimoire contracts` to review external API documentation before designing new uispec endpoint specs.

### Notes

- Grimoire works out of the box after plugin installation — no setup needed
- If Context7 is unavailable, grimoire-docs falls back to web search automatically
- If a research stream finds nothing relevant, it reports gaps rather than fabricating results
- Grimoire adapts to any codebase — agents work with whatever is present
- For full research, provide a descriptive context sentence (e.g., "add JWT authentication with refresh tokens to the Express API")
