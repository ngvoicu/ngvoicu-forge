---
name: runebook-implementation
description: "Core implementation logic for runebook commands. Do not invoke directly — use the /runebook-* commands instead."
---

# Runebook Implementation

This skill contains the core logic for all runebook commands. It is invoked by the thin command wrappers.

## How It Works

All state lives in `.runebook/` at the project root. Entries are organized by type:

```
.runebook/
├── index.md              # master index with links, tags, dates
├── endpoints/
│   └── <name>.md
├── jobs/
│   └── <name>.md
├── flows/
│   └── <name>.md
├── services/
│   └── <name>.md
└── integrations/
    └── <name>.md
```

Each entry is a markdown file with YAML frontmatter containing metadata (type, dates, source files, related entries, tags) and structured documentation sections specific to the component type.

## Error Handling

Before executing any action, check for and handle these edge cases:

- **Missing `.runebook/` directory**: On any action except `init`, check that `.runebook/` exists. If not, error: "No `.runebook/` directory found. Run `/runebook-init` to scan your codebase and create the runebook."
- **Name collisions**: Before creating a new entry, check if a file with that name already exists in the target type directory. If it does, error: "An entry named `<name>` already exists in `<type>/`. Use `/runebook-update <type>/<name>` to update it."
- **Git merge conflicts**: When reading any entry file, check for `<<<<< HEAD` conflict markers. If found, error: "Entry `<name>` has unresolved git merge conflicts. Please resolve them before continuing." Do not attempt to parse or use the conflicted file.
- **Ambiguous names**: When user provides a name without a type prefix and multiple matches exist across types, list all matches and ask the user to specify: "Found `<name>` in multiple types: `endpoints/<name>`, `services/<name>`. Which one?"
- **Invalid names**: Validate that entry names contain only alphanumeric characters, hyphens, and underscores. If the user provides a name with spaces or special characters, auto-slugify it (e.g., "User Auth Service" → "user-auth-service") and confirm with the user before proceeding.
- **Missing source files**: When validating or updating, if a source file referenced in an entry no longer exists on disk, flag it clearly: "Source file `<path>` referenced in `<entry>` no longer exists." Do not silently remove the reference.
- **Index out of sync**: If `index.md` lists entries that don't exist as files, or files exist that aren't in the index, report the discrepancy and suggest running `/runebook-update` to fix.

---

## Action: init

Initialize the runebook by scanning the codebase and generating documentation entries.

### Steps

1. **Check for existing `.runebook/`**
   - If it exists, read `index.md` to understand what's already documented
   - Preserve existing entries — only add missing ones
   - If it doesn't exist, create the full directory structure:
     ```
     .runebook/
     ├── index.md
     ├── endpoints/
     ├── jobs/
     ├── flows/
     ├── services/
     └── integrations/
     ```

2. **Invoke the `runebook-scanner` agent** for a thorough codebase scan
   - Use `Task` tool with `subagent_type: "general-purpose"`
   - Provide the full scanner agent prompt from `runebook/agents/runebook-scanner.md`
   - Agent reads source files in detail — traces route handlers, reads cron definitions, follows service class methods, identifies external API calls
   - Wait for the agent to complete and read its findings

3. **Generate entry files** from scanner results
   - For each discovered component, create a markdown file using the appropriate template (see Entry Templates below)
   - Fill template sections with data from the scanner's analysis
   - Set frontmatter: `type`, `updated` (today's ISO date), `source_files`, `related` (cross-references), `tags`
   - Skip entries that already exist (when re-running init)

4. **Generate `index.md`**
   - Create or update the master index with a table per type:

   ```markdown
   # Runebook Index

   > Living documentation — auto-generated and kept in sync with code.
   > Last updated: <ISO date>

   ## Endpoints

   | Name | Method | Path | Tags | Updated |
   |------|--------|------|------|---------|
   | [<name>](endpoints/<name>.md) | GET | /api/... | auth, users | 2025-01-15 |

   ## Jobs

   | Name | Schedule | Tags | Updated |
   |------|----------|------|---------|
   | [<name>](jobs/<name>.md) | */5 * * * * | cleanup | 2025-01-15 |

   ## Flows

   | Name | Trigger | Tags | Updated |
   |------|---------|------|---------|
   | [<name>](flows/<name>.md) | User signup | onboarding | 2025-01-15 |

   ## Services

   | Name | Methods | Tags | Updated |
   |------|---------|------|---------|
   | [<name>](services/<name>.md) | 5 public | auth, core | 2025-01-15 |

   ## Integrations

   | Name | Provider | Tags | Updated |
   |------|----------|------|---------|
   | [<name>](integrations/<name>.md) | Stripe | payments | 2025-01-15 |
   ```

5. **Ask about flows**
   - Flows (multi-component business processes) cannot be fully auto-detected
   - Present any potential flows the scanner found and ask: "The scanner identified these potential flows. Would you like me to create entries for any of them? You can also describe additional business flows."

6. **Report results**
   ```
   Runebook initialized:
   - Endpoints: 12 created
   - Jobs: 3 created
   - Services: 8 created
   - Integrations: 4 created
   - Flows: 0 (awaiting your input)
   - Skipped: 2 (already existed)
   - TODOs: 3 entries need human context (marked with TODO)
   ```

---

## Action: update

Update runebook entries after code changes.

### With name argument: `<type>/<name>` or `<name>`

1. **Resolve the entry**
   - If given `<type>/<name>`, read `.runebook/<type>/<name>.md` directly
   - If given just `<name>`, search all type directories for a match
   - If multiple matches, list them and ask user to specify
   - If no match, error with suggestion to use `/runebook-init`

2. **Re-read source files**
   - Read each file listed in the entry's `source_files` frontmatter
   - Identify what changed compared to the documented behavior

3. **Update the entry**
   - Update relevant sections (behavior, dependencies, request/response shapes, etc.)
   - Preserve human-written context — notes, explanations, custom sections
   - Update frontmatter: `updated` date, add/remove `source_files` if changed, update `tags`
   - Append to the changelog:
     ```markdown
     ## Changelog

     ### <ISO date>
     - <what changed and why>
     ```
   - Update cross-references: if this entry's dependencies changed, update `related` in both this entry and the related entries (bidirectional)

4. **Update `index.md`** if any metadata changed (method, path, schedule, tags, etc.)

### Without name argument: auto-detect from recent changes

1. **Get recent changes**
   - Run `git diff --name-only HEAD~1` (or `git diff --name-only --cached` if there are staged changes)
   - Filter to source files only (exclude tests, configs, lockfiles, docs)

2. **Invoke the `runebook-scanner` agent** with scoped scan
   - Pass the list of changed files
   - Agent reads each changed file, compares against existing entries, identifies what's stale

3. **Apply updates**
   - For each stale entry: update the entry body, frontmatter, and changelog
   - For new undocumented components: create entries using templates
   - For removed components: flag but don't delete — ask user for confirmation

4. **Report**
   ```
   Runebook updated:
   - Updated: endpoints/create-user, services/auth-service
   - Created: endpoints/reset-password (new)
   - Flagged: jobs/cleanup-temp (source file removed)
   ```

### Update rules

- **Preserve human context**: Never overwrite sections that contain human-written notes, explanations, or annotations not derivable from code
- **Changelog is append-only**: Never remove or modify existing changelog entries
- **Cross-references are bidirectional**: When adding `related: services/auth` to an endpoint, also add `related: endpoints/login` to the service
- **Source files are verified**: Before updating, confirm all `source_files` still exist. Flag missing ones.

---

## Action: validate

Check the runebook for staleness, missing coverage, broken references, and inconsistencies.

### Checks

1. **Staleness check**
   - For each entry, compare its `updated` date against the last modified date of each `source_files` entry:
     ```bash
     git log -1 --format=%aI -- <filepath>
     ```
   - If source was modified after the entry's `updated` date, flag as stale

2. **Missing documentation check**
   - Invoke the scanner agent in discovery mode (or use Glob/Grep for common patterns)
   - Compare discovered components against existing entries
   - Report undocumented components

3. **Broken cross-references**
   - For each entry, verify every item in `related` frontmatter:
     - The referenced file exists
     - The referenced file also references back (bidirectional check)
   - Report one-way or broken references

4. **Source file integrity**
   - For each entry, verify every item in `source_files` frontmatter still exists on disk
   - Report missing files

5. **Index consistency**
   - Compare `index.md` entries against actual files in type directories
   - Report: files in index but not on disk, files on disk but not in index

### Output

```
Runebook Validation Report
==========================

Coverage: 34/38 components documented (89%)

Stale entries (4):
  ! endpoints/create-user — source modified 2025-01-20, entry updated 2025-01-10
  ! services/payment-service — source modified 2025-01-19, entry updated 2025-01-05
  ...

Missing documentation (4):
  + POST /api/v2/webhooks/stripe (src/routes/webhooks.ts:45)
  + CleanupWorker (src/workers/cleanup.ts:12)
  ...

Broken references (1):
  ~ endpoints/login → services/old-auth (file does not exist)

Missing source files (0):
  (none)

Index consistency:
  - index.md lists jobs/legacy-sync but file not found
  + services/cache-service exists but not in index.md

Suggestions:
  - Run /runebook-update to fix stale entries
  - Run /runebook-update to add missing entries
  - Manually review broken references
```

---

## Action: show

Display a runebook entry or the index.

### With name argument

1. Resolve the entry (same logic as update — handle `<type>/<name>` or `<name>`, disambiguate if needed)
2. Read `.runebook/<type>/<name>.md`
3. Display the full contents

### Without name argument

1. Read `.runebook/index.md`
2. Display the full contents

---

## Action: status

Show a dashboard overview of the runebook.

### Steps

1. Read `.runebook/index.md` for the master list
2. Glob all entry files across type directories
3. For each entry, read frontmatter to extract: type, updated date, tags, source_files count
4. Check staleness (compare `updated` vs git last-modified of source files)
5. Display dashboard:

```
Runebook Status
===============

                    Documented    Stale    Coverage
  Endpoints              12        2       12/14 (86%)
  Jobs                    3        0        3/3 (100%)
  Flows                   2        0          2 (manual)
  Services                8        1        8/9 (89%)
  Integrations            4        0        4/4 (100%)
  ─────────────────────────────────────────────────
  Total                  29        3       29/32 (91%)

Recently Updated:
  endpoints/create-user          2025-01-20    auth, users
  services/payment-service       2025-01-19    payments, stripe
  integrations/stripe-webhooks   2025-01-18    payments, webhooks

Tags:
  auth (8)  payments (5)  users (4)  core (3)  admin (2)  ...

Stale (need /runebook-update):
  ! endpoints/login — 10 days behind source
  ! endpoints/register — 5 days behind source
  ! services/email-service — 3 days behind source
```

---

## Entry Templates

### Endpoint Template

```markdown
---
type: endpoint
method: <METHOD>
path: <path>
updated: <ISO date>
source_files:
  - <file path>
related:
  - services/<name>
tags:
  - <tag>
---

# <METHOD> <path>

## Summary

<One-paragraph description of what this endpoint does and when it's used.>

## Authentication & Authorization

- **Auth required:** <yes/no>
- **Method:** <Bearer token / API key / Session / None>
- **Roles:** <admin, user, public — who can access>

## Request

### Parameters

| Name | In | Type | Required | Description |
|------|----|------|----------|-------------|
| id | path | string | yes | ... |

### Body

\`\`\`json
{
  "field": "type — description"
}
\`\`\`

### Validation Rules

- <field>: <rule>

## Response

### Success (<status code>)

\`\`\`json
{
  "field": "type — description"
}
\`\`\`

### Errors

| Status | Code | Description | When |
|--------|------|-------------|------|
| 400 | VALIDATION_ERROR | Invalid input | ... |
| 401 | UNAUTHORIZED | Missing/invalid auth | ... |
| 404 | NOT_FOUND | Resource not found | ... |

## Behavior

### Happy Path

1. <Step-by-step description of what happens on a successful request>

### Edge Cases

- <Edge case and how it's handled>

## Dependencies

- **Services:** <list of services called>
- **Integrations:** <external APIs called>
- **Database:** <tables/collections accessed>

## Changelog

### <ISO date>
- Initial documentation from codebase scan
```

### Job Template

```markdown
---
type: job
schedule: "<cron expression or trigger>"
updated: <ISO date>
source_files:
  - <file path>
related:
  - services/<name>
tags:
  - <tag>
---

# <Job Name>

## Summary

<One-paragraph description of what this job does and why it exists.>

## Schedule

- **Cron:** <expression> (<human-readable>)
- **Timezone:** <timezone or UTC>
- **Queue:** <queue name if applicable>
- **Concurrency:** <can multiple instances run? mutex/lock?>

## Execution

### Steps

1. <What happens first>
2. <What happens next>
3. ...

### Input

- <What data does the job consume? How does it get it?>

### Output

- <What does the job produce? Side effects?>

## Retry & Failure Handling

- **Max retries:** <n>
- **Backoff:** <strategy>
- **Dead letter:** <where do failed jobs go?>
- **On failure:** <what happens — alert, skip, compensate?>

## Monitoring

- <How to know the job ran successfully>
- <Alerting for failures>
- <Expected duration / SLA>

## Dependencies

- **Services:** <list>
- **Integrations:** <list>
- **Database:** <tables accessed>

## Changelog

### <ISO date>
- Initial documentation from codebase scan
```

### Flow Template

```markdown
---
type: flow
trigger: "<what starts this flow>"
updated: <ISO date>
source_files:
  - <file path>
related:
  - endpoints/<name>
  - services/<name>
tags:
  - <tag>
---

# <Flow Name>

## Summary

<One-paragraph description of this business flow — what it accomplishes end-to-end.>

## Trigger

<What initiates this flow? User action, event, schedule, webhook?>

## Steps

| # | Component | What Happens | Can Fail? | On Failure |
|---|-----------|-------------|-----------|------------|
| 1 | endpoints/signup | Receives user data, validates | Yes | Return 400 |
| 2 | services/auth | Creates user account | Yes | Return 500, log |
| 3 | integrations/sendgrid | Sends welcome email | Yes | Queue retry, continue |
| 4 | services/onboarding | Creates default workspace | Yes | Flag for manual setup |

## Completion Criteria

<What must be true for this flow to be considered "done"?>

## Error Handling

| Step | Error | Recovery |
|------|-------|----------|
| 2 | Duplicate email | Return 409, suggest login |
| 3 | Email service down | Queue for retry, don't block signup |

## Data Flow

<What data moves between steps? Transformations?>

## Side Effects

- <External systems affected>
- <Events emitted>
- <Notifications sent>

## Changelog

### <ISO date>
- Initial documentation
```

### Service Template

```markdown
---
type: service
updated: <ISO date>
source_files:
  - <file path>
related:
  - endpoints/<name>
tags:
  - <tag>
---

# <Service Name>

## Summary

<One-paragraph description of this service's responsibility in the system.>

## Public Interface

### `<methodName>(params) → returnType`

<Description of what this method does.>

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| ... | ... | ... | ... |

**Returns:** <type> — <description>

**Throws:**
- `<ErrorType>` — <when>

---

### `<nextMethod>(params) → returnType`

...

## Internal Logic

<Key algorithms, decision trees, state machines — the "how" that isn't obvious from the interface.>

## Dependencies

- **Services:** <other services used>
- **Integrations:** <external APIs called>
- **Database:** <tables/collections accessed>
- **Config:** <environment variables / feature flags>

## State Management

<Does this service maintain state? Cache? In-memory data? How is it invalidated?>

## Changelog

### <ISO date>
- Initial documentation from codebase scan
```

### Integration Template

```markdown
---
type: integration
provider: "<provider name>"
updated: <ISO date>
source_files:
  - <file path>
related:
  - services/<name>
tags:
  - <tag>
---

# <Integration Name>

## Summary

<One-paragraph description of what this integration does and why.>

## Provider

- **Service:** <provider name (e.g., Stripe, SendGrid, AWS S3)>
- **Documentation:** <link to provider docs>
- **Dashboard:** <link to provider dashboard>

## Authentication

- **Method:** <API key / OAuth / Service account>
- **Credentials:** <env var names — NOT values>
- **Rotation:** <how to rotate credentials>

## API Calls Made

### `<operation name>`

- **Method:** <HTTP method>
- **Endpoint:** <URL pattern>
- **When:** <what triggers this call>
- **Request:** <key fields>
- **Response:** <what we use from the response>

---

### `<next operation>`

...

## Webhooks Received

### `<event type>`

- **Path:** <our webhook endpoint>
- **Payload:** <key fields>
- **Action:** <what we do when received>
- **Verification:** <how we verify authenticity>

## Error Handling

- **Timeout:** <value and what happens>
- **Retries:** <strategy>
- **Circuit breaker:** <threshold and behavior>
- **Fallback:** <what happens when the integration is down>

## Data Mapping

| Our Field | Provider Field | Transform |
|-----------|---------------|-----------|
| userId | customer_id | none |
| amount | amount | cents → dollars |

## Changelog

### <ISO date>
- Initial documentation from codebase scan
```

---

## Index Management

The `index.md` file is the master reference for the runebook. It must always be in sync with the actual entry files.

### When to update index.md

- After creating new entries (init, update without name)
- After updating entries that change metadata (method, path, schedule, tags)
- After deleting entries
- Never: when only the entry body changes (behavior descriptions, notes)

### Index format

See the template in Action: init, step 4.

---

## Critical Behaviors

- **Scanner agent is thorough** — it reads source files, not just file names. Quality depends on this.
- **Entries are living documents** — they evolve with the code, not just at init time
- **Human context is sacred** — never overwrite explanations, notes, or context a human added
- **Changelog is append-only** — the history of changes is as valuable as the current state
- **Cross-references are bidirectional** — if A references B, B must reference A
- **Source files are the source of truth** — when code and docs disagree, code wins, docs get updated
- **Flows need humans** — auto-detection finds components, but business flows require human knowledge
- **Commit `.runebook/` to git** — documentation must survive across sessions
- **Tags enable discovery** — consistent tagging helps find related components across types
