## Runebook (MANDATORY)

Living documentation lives in `.runebook/`. **ALWAYS consult before changes, ALWAYS update after changes — automatically, without asking.**

### When to Use Runebook

| Situation | Action |
|-----------|--------|
| Before modifying any code | Read relevant `.runebook/<type>/<name>.md` entries |
| After creating a new endpoint | Create `.runebook/endpoints/<name>.md` |
| After creating a new scheduled job | Create `.runebook/jobs/<name>.md` |
| After creating a new service/module | Create `.runebook/services/<name>.md` |
| After creating a new integration | Create `.runebook/integrations/<name>.md` |
| After creating a new page/route | Create `.runebook/pages/<name>.md` |
| After creating a new reusable component | Create `.runebook/components/<name>.md` |
| After creating a new custom hook | Create `.runebook/hooks/<name>.md` |
| After modifying any existing component | Update the existing entry |
| After removing a component | Flag entry as removed (add `status: removed` to frontmatter) |
| New business flow identified | Create `.runebook/flows/<name>.md` (ask about steps) |

### Before ANY Code Changes

You MUST read relevant runebook entries before modifying code:
1. Identify which components will be affected by your changes
2. Read their entries from `.runebook/<type>/<name>.md`
3. Understand current behavior, dependencies, edge cases, and recent changes
4. Use this context to inform your implementation

**How to find the right entries:** Match changed files to runebook entries via `source_files` in frontmatter, or search by tag/name in `.runebook/index.md`.

### After ANY Code Changes

You MUST update the runebook after changes — do it automatically, do not ask:

For every update:
- Update the entry's behavior, dependencies, and request/response sections as needed
- Preserve human-written notes and context — **never overwrite them**
- Update frontmatter: `updated` date, `source_files`, `tags`
- Append to the changelog with today's date and what changed — **changelog is append-only**
- Update cross-references bidirectionally (if A depends on B, both entries reference each other)
- Update `.runebook/index.md` if metadata changed (new entries, changed methods/paths/schedules)

### Rules

- **ALWAYS read entries before code work** — never modify code without checking runebook first
- **ALWAYS update entries after changes** — automatically, without asking permission
- **NEVER overwrite human context** — notes, explanations, and annotations are sacred
- **NEVER modify existing changelog entries** — changelog is append-only
- **ALWAYS update cross-references bidirectionally** — if entry A references B, entry B must reference A

### Naming Conventions

| Type | File Name Pattern | Example |
|------|-------------------|---------|
| Endpoints | `<method>-<path-slug>.md` | `post-users.md`, `get-orders-by-id.md` |
| Jobs | `<job-name>.md` | `cleanup-expired-sessions.md` |
| Services | `<service-name>.md` | `auth-service.md`, `payment-processor.md` |
| Integrations | `<provider-name>.md` | `stripe.md`, `sendgrid.md` |
| Flows | `<flow-name>.md` | `user-onboarding.md`, `order-checkout.md` |
| Pages | `<route-path-slug>.md` | `users-by-id.md`, `dashboard.md` |
| Components | `<component-name>.md` | `button.md`, `data-table.md` |
| Hooks | `<hook-name>.md` | `use-auth.md`, `use-pagination.md` |

### First-Time Setup

If `.runebook/` doesn't exist yet, run `/runebook-init` to scan the codebase and create the runebook.

### Commands

```
/runebook-init              # Scan codebase, create .runebook/
/runebook-update [name]     # Update specific entry or auto-detect from changes
/runebook-validate          # Check staleness, coverage, broken refs
/runebook-show [name]       # Display entry or master index
/runebook-status            # Dashboard with coverage and staleness
```

### Integration with Other Plugins

**With specsmith:** When implementing a spec, read relevant runebook entries during implementation to understand components being modified. After each step, update affected entries.

**With uispec:** When syncing UI specs after API changes, ensure runebook endpoint entries are consistent with the API documentation in `uispec/`.

**With grimoire:** Use `/grimoire codebase` to understand code before creating runebook entries for complex components. Runebook entries provide useful context for grimoire research.

**With chisel:** After `/chisel rename` or `/chisel move` operations, update runebook entries whose `source_files` reference the affected paths.

### Notes

- If `.runebook/` doesn't exist, run `/runebook-init` to create it
- If an entry has git merge conflicts, resolve them before continuing
- If a source file referenced in an entry no longer exists, flag it — don't silently remove the reference
- If `index.md` is out of sync with actual files, run `/runebook-validate` to detect and fix
