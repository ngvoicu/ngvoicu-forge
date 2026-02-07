# ngvoicu-forge-claude-code

Claude Code plugin marketplace with six plugins for spec-driven development, deep research, living documentation, surgical code editing, and UI specification workflows. All plugins are markdown-based — no build system, no dependencies.

## Quick Start

```bash
# Add the marketplace
/plugin marketplace add https://github.com/ngvoicu/ngvoicu-forge-claude-code.git

# Install what you need
/plugin install specsmith@ngvoicu-forge-claude-code
/plugin install grimoire@ngvoicu-forge-claude-code
/plugin install runebook@ngvoicu-forge-claude-code
/plugin install chisel@ngvoicu-forge-claude-code
/plugin install uispec-backend@ngvoicu-forge-claude-code
/plugin install uispec-frontend@ngvoicu-forge-claude-code
```

---

## The Complete Development Workflow

All six plugins form a complete development pipeline — from planning a feature to shipping it with full documentation, tested code, and synced API contracts. Here's how they work together.

### The Problem

You're building a feature. Requirements live in someone's head. Nobody researches best practices before coding. The backend changes an endpoint, the frontend doesn't know. A file gets renamed, half the imports break. Documentation is stale or doesn't exist. By the time you notice, the bug is in production and nobody remembers why things were built this way.

### The Solution

Each plugin handles one piece of the development lifecycle:

| Plugin | Role | What it produces |
|--------|------|-----------------|
| **specsmith** | Planning | Implementation specs with phased steps |
| **grimoire** | Research | Library docs, best practices, codebase analysis |
| **runebook** | Documentation | Living docs for every component + system guides |
| **chisel** | Code editing | Safe renames, moves, and replacements |
| **uispec-backend** | API contract | Synced `openapi.yaml` and endpoint specs |
| **uispec-frontend** | UI implementation | Generated UI from specs with validation |

### Step-by-Step: From Idea to Shipped Feature

#### 1. Spec the feature (specsmith)

```
/specsmith-new auth-refactor: Refactor the auth middleware to support both JWT and session tokens
```

Specsmith runs a discovery interview, researches your codebase, and produces a full spec with phased implementation steps. The spec includes API design — endpoint paths, request/response shapes, error codes, auth requirements.

Behind the scenes, if grimoire is installed, specsmith uses its specialized agents for research — library docs via Context7, best practices, codebase analysis, git history, DB schema, and external API contracts. All running in parallel.

#### 2. Research before building (grimoire)

Even outside specsmith, use grimoire whenever you're about to write code and you're not 100% sure of the approach. Just describe what you need — grimoire picks the right agents automatically:

```bash
# Grimoire auto-dispatches to the right agents based on your query:
/grimoire add JWT refresh token rotation to the auth middleware
/grimoire how does Stripe Connect handle marketplace payouts

# Or use a specific stream for a quick focused lookup:
/grimoire docs jsonwebtoken verify RS256 tokens
```

Grimoire feeds into specsmith Phase 2 automatically, but you can also use it standalone any time you need grounded research instead of guessing.

#### 3. Check existing documentation (runebook)

Before you start coding, runebook tells you how the system currently works:

```bash
# Read what's documented about the area you're changing:
/runebook-show auth-service

# Or read the full system guide for the domain:
/runebook-show authentication
```

Runebook entries cover what each component does, its dependencies, edge cases, and recent changes. System guides give you the big-picture narrative — how authentication works end-to-end, not just individual endpoints.

If runebook isn't set up yet, initialize it first:

```
/runebook-init
```

This scans your entire codebase and generates entries for every endpoint, service, job, flow, integration, page, component, and hook — plus narrative system guides for each feature domain.

#### 4. Implement the backend (specsmith drives)

```
/specsmith-implement
```

Specsmith walks you through each step. As you implement:

- **Runebook watches automatically** — after each code change, it updates the affected entries and guides without asking
- **Specsmith reminds you to sync** — when a step creates or modifies an API endpoint:

> "This step changed API endpoints. Run `/uispec-sync` to update the API contract."

#### 5. Rename or restructure safely (chisel)

If implementation requires renaming files, moving code, or replacing patterns across the codebase:

```bash
# Rename a module — updates all imports, tests, stories:
/chisel rename AuthMiddleware to SessionAuthMiddleware

# Move files to a new structure:
/chisel move src/utils/jwt.ts to src/auth/jwt.ts

# Replace a function name across the codebase:
/chisel replace verifyToken with validateSession
```

Chisel preserves git history, finds co-located tests and barrel exports, and updates runebook entries referencing affected files — all automatically.

#### 6. Sync the API contract to frontend (uispec-backend)

```
/uispec-sync
```

UISpec scans your backend, detects what changed, and updates the frontend project's `uispec/` directory:

- `openapi.yaml` gets the new/modified endpoints
- `endpoints/<name>.md` files are created or updated with request/response shapes, auth rules, error codes
- Frontend-owned sections (UI guidelines, design system) are preserved — never overwritten

#### 7. Build the frontend from the spec (uispec-frontend)

```
/uispec-implement create-user
```

The frontend plugin reads the endpoint spec, your design system tokens, and your component library docs, then generates the UI implementation — with proper loading states, error handling, validation, and accessibility. It follows your project's conventions (React, Vue, Angular, whatever it finds).

#### 8. Validate everything

```bash
# Backend: are all endpoints documented? Any orphaned specs?
/uispec-validate

# Frontend: does the UI match the spec? Missing states? Wrong shapes?
/uispec-validate-ui
/uispec-detect

# Documentation: is everything in sync? Anything stale?
/runebook-validate
```

#### 9. Iterate

When requirements change, go back to specsmith:

```
/specsmith-edit auth-refactor
```

Update the spec, implement the changes, run `/uispec-sync` again. Runebook updates automatically. The contract stays in sync. The documentation stays current. The frontend knows exactly what changed.

### Updating Existing Endpoints

The flow above covers new features. But most of the time you're changing something that already exists — adding a field, tweaking validation, changing a response shape. Here's the update flow:

#### 1. Research first (grimoire)

Before changing anything, understand the current state:

```bash
# Grimoire auto-picks the right agents:
/grimoire how does user registration currently work and what are the edge cases
```

#### 2. Check the documentation (runebook)

```bash
# Read the entry for the endpoint you're changing:
/runebook-show create-user

# Read the domain guide for context:
/runebook-show user-onboarding
```

#### 3. Make the backend change

Modify the endpoint code as needed — add a field, change auth, update error handling. Runebook updates the affected entries automatically after your changes.

#### 4. Sync the updated contract

```
/uispec-sync
```

UISpec detects what changed, updates the API sections in `endpoints/<name>.md`, and appends a changelog entry. Frontend-owned sections (UI guidelines, design tokens) are preserved.

#### 5. See what's out of sync

```
/uispec-detect
```

Detection shows the gaps: shape mismatches (`response missing emailVerified field`), new error states not handled, undocumented API calls. This tells you exactly what the frontend needs to catch up on.

#### 6. Implement with context

```
/uispec-implement create-user: Added email verification — show inline validation and a pending-verification state after submit
```

The colon syntax works like specsmith — endpoint name before the colon, context after it. The context tells uispec **why** things changed and **how** you want the UI to handle it. Without it, uispec implements the spec changes mechanically. With it, you guide the implementation decisions.

Some examples:

```
/uispec-implement login: Session tokens replaced JWT — remove token refresh logic, add cookie-based auth
/uispec-implement get-products: Added pagination — use infinite scroll, not page numbers
/uispec-implement create-order: New discount_code field — show collapsible promo section, validate format client-side
```

You can also run without context if the spec changes are self-explanatory:

```
/uispec-implement create-user
```

#### 7. Validate

```bash
/uispec-validate-ui   # UI matches updated spec
/runebook-validate     # documentation is current
```

### What Each Plugin Owns

| | Backend-owned | Frontend-owned |
|---|---|---|
| **API contract** | `openapi.yaml`, Method/Path/Auth/Request/Response sections | Read-only |
| **Suggested implementation** | Backend generates (component suggestions, interactions, watchouts) | Can override in UI Guidelines |
| **UI guidelines** | Read-only | `design-system.md`, `components.md`, UI Guidelines sections |
| **Implementation spec** | `.specsmiths/` — full spec with phased steps | Can read, doesn't modify |
| **Documentation** | `.runebook/` — entries + guides (shared, both sides update) | `.runebook/` — entries + guides (shared, both sides update) |
| **Changelog** | Both append (append-only, never modify existing entries) | Both append |

### When to Use What

| You're about to... | Use this |
|---|---|
| Build a new feature | `/specsmith-new <name>: <brief>` |
| Research before coding | `/grimoire <query>` (auto-dispatches to relevant agents) |
| Understand existing code before changing it | `/grimoire <query>` and `/runebook-show <name>` |
| Rename or move files | `/chisel rename` or `/chisel move` |
| Create or modify a backend endpoint | Code it, then `/uispec-sync` |
| Build UI for an endpoint | `/uispec-implement <endpoint>` |
| Update UI after a backend change | `/uispec-implement <endpoint>: <what changed>` |
| Check if anything is out of sync | `/uispec-detect`, `/uispec-validate-ui`, `/runebook-validate` |
| Onboard to unfamiliar code | `/runebook-show` for entries, `/runebook-show <guide>` for domain guides |

---

## Plugins

### specsmith

Research-driven implementation specs. You describe what you want to build, specsmith researches your codebase and the ecosystem, then produces a full spec with phased implementation steps.

#### How it works

Every `/specsmith-new` triggers a three-phase process:

1. **Discovery** — Provide a name and brief in one shot. Specsmith checks if your brief has enough substance (specific behavior, a trigger or problem, not just a vague phrase). If the brief is comprehensive, it moves straight to research. If not, it asks focused follow-up questions — edge cases, constraints, flows. All Q&A is logged to `.specsmiths/<n>.questions.md`.
2. **Deep research** — Adaptive subagents (1-6) investigate based on what discovery revealed: codebase analysis, library docs, best practices, git history, DB schema, external APIs. If grimoire is installed, specsmith uses its specialized agents for richer research. Findings go to `.specsmiths/<n>.research.md`.
3. **Spec creation** — A full specification with requirements, architecture, data model, API design, edge case table, implementation steps (each with What/Where/How/Verify), risks, testing strategy, and rollback plan.

The result is a spec you could hand to any developer and they'd know exactly what to build. All state lives in `.specsmiths/` as plain markdown — commit it to git.

#### Typical workflow

```bash
# Create a spec — one-shot with name and brief
/specsmith-new auth-refactor: Refactor the auth middleware to support both JWT and session tokens

# Specsmith runs discovery, researches your codebase, and creates the spec.
# Review the spec, then start implementing:
/specsmith-implement

# If you get interrupted, resume later — specsmith re-reads everything:
/specsmith-resume

# Requirements changed mid-flight? Edit the spec:
/specsmith-edit auth-refactor

# Working on multiple features? Switch between specs:
/specsmith-switch payments-v2

# Check the status of all your specs:
/specsmith-status
```

For simple changes touching 1-2 files with existing dependencies, you can request lightweight research: just tell specsmith "keep it light" or "skip the deep research" and it will limit to codebase analysis only.

#### Commands

```
/specsmith-new <name>: <brief>  # one-shot: discovery -> research -> full spec
/specsmith-new <name>           # name only: asks for brief next
/specsmith-implement            # start active spec implementation
/specsmith-resume               # re-read everything, continue active spec
/specsmith-switch <name>        # save progress, switch active spec
/specsmith-edit <name>          # revise spec mid-flight
/specsmith-status               # dashboard of all specs with phase progress
/specsmith-list                 # interactive list with picker
/specsmith-show [name]          # display spec without activating
/specsmith-archive <name>       # shelve completed/paused spec
/specsmith-unarchive <name>     # restore archived spec
/specsmith-drop <name>          # delete spec + research + questions
```

### grimoire

Research suite with 6 specialized agents. Just describe what you need — grimoire analyzes your query and picks the right agents automatically. No need to specify which stream to use.

#### How it works

Grimoire has 6 agents: library docs (Context7 + web fallback), best practices, codebase analysis, git history, DB schema, and external API contracts. When you run `/grimoire <query>`, it reads your query, picks 1-6 relevant agents based on the context, runs them all in parallel, and returns a synthesized report.

For example, `/grimoire add user authentication` might spawn codebase analysis + best practices + library docs + git history — all at once.

#### Typical workflow

```bash
# Just describe what you need — grimoire auto-dispatches:
/grimoire add JWT refresh token rotation to the auth middleware
/grimoire how does Stripe Connect handle marketplace payouts
/grimoire why is the payment service timing out

# For a quick single-stream lookup, prefix with the stream name:
/grimoire docs zod how to validate nested objects
/grimoire practices CSRF protection for REST APIs
/grimoire codebase payment processing
/grimoire history websocket migration
/grimoire schema users table indexes
/grimoire contracts stripe webhooks
```

Grimoire is what specsmith Phase 2 uses automatically for research when installed.

#### Commands

```
/grimoire <query>                  # auto-dispatch to relevant agents (default)
/grimoire docs <lib> <query>       # library docs only
/grimoire practices <topic>        # best practices only
/grimoire codebase <area>          # codebase analysis only
/grimoire history <area>           # git history only
/grimoire schema <area>            # DB schema only
/grimoire contracts <service>      # external API docs only
/grimoire                          # show available streams
```

### runebook

Living documentation that tracks how your application works. Runebook maintains two kinds of documentation:

**Entries** — structured docs for individual components: endpoints, jobs, flows, services, integrations, pages, components, and hooks. Each entry covers what the component does, how it works, its dependencies, edge cases, and recent changes.

**System guides** — narrative markdown documents organized by feature domain (authentication, payments, onboarding) that explain how things work end-to-end. Unlike entries, guides read like a walkthrough written by a senior engineer onboarding a new teammate. They're auto-generated from component relationships and kept in sync as code changes, while preserving any human-written sections (tribal knowledge, gotchas, common tasks).

Runebook is **assertive**: once installed, it reads entries and guides before code work and updates them after changes — automatically, without asking.

#### Typical workflow

```bash
# First time: scan the entire codebase and generate everything
/runebook-init

# Runebook creates:
#   .runebook/index.md              — master index
#   .runebook/endpoints/login.md    — entry for each endpoint
#   .runebook/services/auth.md      — entry for each service
#   .runebook/guides/authentication.md — guide for the auth domain
#   ... etc.

# After making changes, update a specific entry:
/runebook-update login

# Or let runebook auto-detect what changed:
/runebook-update

# Check if anything is stale, missing, or broken:
/runebook-validate

# Quick look at an entry or the full index:
/runebook-show login
/runebook-show

# Dashboard with coverage stats, staleness, and guide health:
/runebook-status
```

#### How guides work

Guides are generated from component clusters — when 3+ entries across 2+ types share tags, imports, or serve the same user flow, runebook identifies that as a feature domain and writes a narrative guide for it.

Guide sections are split into two categories. **Auto-derivable sections** (How It Works, Architecture at a Glance, Where Things Live) are updated from code whenever entries change. **Human sections** (Key Concepts, Common Tasks, Gotchas & Tribal Knowledge) are preserved always — runebook never overwrites what you write there.

#### Commands

```
/runebook-init              # scan codebase, create .runebook/ with entries + guides
/runebook-update [name]     # update specific entry or auto-detect from changes
/runebook-validate          # check staleness, coverage, broken refs, guide freshness
/runebook-show [name]       # display entry or master index
/runebook-status            # dashboard with coverage, staleness, and guide health
```

### chisel

Fast surgical code editor for renaming, moving, replacing, and making small targeted edits. Uses the haiku model for speed. Never manually rename or move files — chisel handles all reference updates across your entire codebase automatically.

#### Why use chisel instead of find-and-replace

Chisel is **git-aware** (uses `git mv` to preserve file history), **framework-aware** (finds co-located tests, CSS modules, stories, barrel exports), **scope-aware** (respects word boundaries, warns on large scope, previews changes), and **language-agnostic** (any language, framework, or file type).

A manual rename misses test files, barrel exports, and dynamic imports. Chisel catches them all.

#### Typical workflow

```bash
# Rename a component — updates all imports, tests, stories, CSS modules:
/chisel rename UserCard to ProfileCard

# Move a file to a new directory — updates all import paths:
/chisel move src/utils/auth.ts to src/services/auth.ts

# Replace a string across the codebase with scope analysis:
/chisel replace getUser with fetchUser

# Small targeted code change described in natural language:
/chisel edit add a loading prop to the Button component

# Or just describe what you want — chisel auto-detects the operation:
/chisel move the auth helpers into the services folder
```

After chisel operations, runebook entries referencing affected files are updated automatically.

#### Commands

```
/chisel                                # show available operations
/chisel rename <old> to <new>          # rename + update all references
/chisel move <source> to <dest>        # move + update all import paths
/chisel replace <old> with <new>       # find-and-replace with scope analysis
/chisel edit <description>             # small targeted code change
/chisel <natural language>             # auto-detect operation
```

### uispec-backend

Backend plugin for maintaining the API contract. After any endpoint change, syncs `openapi.yaml`, endpoint spec files, and `components.md` in the frontend project's `uispec/` directory.

#### How it works

UISpec Backend scans your backend routes, detects what changed, and pushes updates to the frontend's `uispec/` directory. It enforces strict ownership — backend owns the API contract (method, path, auth, request/response shapes), frontend owns the UI sections (design system, components, guidelines). Neither side overwrites the other.

#### Typical workflow

```bash
# First time: generate uispec/ from all existing endpoints
/uispec-init

# After any API change (new endpoint, shape change, auth change):
/uispec-sync

# Check that all endpoints are documented and specs are consistent:
/uispec-validate
```

The `uispec/` directory is created in the frontend project, not the backend. Configure the path in your backend CLAUDE.md (replace `{{PRJ-frontend}}` with your frontend directory name).

#### What gets synced

Each `/uispec-sync` updates four things in the frontend:

1. `openapi.yaml` — the OpenAPI specification with paths, schemas, and auth
2. `endpoints/<name>.md` — per-endpoint spec with request/response shapes, error codes, auth rules
3. `components.md` — shared UI component patterns (only if new patterns are needed)
4. **Suggested Implementation** section in each endpoint file — backend-generated component suggestions, key interactions, and things to watch out for (rate limits, file uploads, nested objects, etc.)

The Suggested Implementation section gives the frontend team a head start — it infers the endpoint type (create-form, list-view, detail-view, delete-action) and suggests components, interactions, and states. Frontend can override any of it in UI Guidelines.

Frontend-owned sections (UI Guidelines, States, Validation, Accessibility) are never touched.

#### Commands

```
/uispec-init      # initialize uispec/ from existing backend endpoints
/uispec-sync      # sync after backend API changes
/uispec-validate  # check spec consistency and coverage
```

### uispec-frontend

Frontend plugin for building UI from specs. Reads the API contract and design system from `uispec/`, detects gaps between specs and your implementation, and generates or updates UI code following your project's conventions.

#### How it works

UISpec Frontend knows which sections are backend-owned (API contract — read only) and which are frontend-owned (design system, components, UI guidelines — editable). It reads the spec, your design tokens, and your component library, then generates UI code with proper loading states, error handling, validation, and accessibility.

#### Typical workflow

```bash
# See what's out of sync — missing UI for specs, API calls without specs:
/uispec-detect

# Build UI for a new endpoint (reads spec, design system, components):
/uispec-implement create-user

# Update UI after a backend change — provide context about what changed:
/uispec-implement create-user: Added email verification — show inline validation and pending state

# Validate that UI matches the spec — PASS or FAIL with details:
/uispec-validate-ui
```

#### The colon syntax

When updating existing endpoints, the colon syntax tells uispec **why** things changed and **how** you want the UI to handle it. Without context, uispec implements the spec diff mechanically. With context, you guide the decisions:

```bash
/uispec-implement login: Session tokens replaced JWT — remove token refresh, add cookie auth
/uispec-implement get-products: Added pagination — use infinite scroll, not page numbers
/uispec-implement create-order: New discount_code field — collapsible promo section
```

#### What `/uispec-detect` checks

1. **Unimplemented specs** — endpoint specs with no UI implementation
2. **Undocumented API calls** — API calls in code with no corresponding spec
3. **Shape mismatches** — request/response fields in code that don't match the spec
4. **Design token violations** — hardcoded colors/spacing instead of design system tokens
5. **Missing state handling** — loading, error, empty, or success states not implemented

#### Commands

```
/uispec-detect                                   # find gaps between specs and UI
/uispec-implement <endpoint>                     # build/update UI from endpoint spec
/uispec-implement <endpoint>: <context>          # build/update with implementation guidance
/uispec-validate-ui                              # validate UI follows spec (PASS/FAIL)
```

---

## How Plugins Work Together

```
specsmith ──→ grimoire     Phase 2 uses grimoire agents for research
specsmith ──→ runebook     Read entries during implementation, update after
specsmith ──→ uispec       After API steps, sync contract to frontend
runebook  ──→ chisel       After rename/move, update runebook source_files
uispec    ──→ runebook     Keep endpoint docs consistent across both
grimoire  ──→ chisel       Research scope before large renames
chisel    ──→ uispec       Update spec files when renaming endpoints
```

---

## General CLAUDE.md Guidelines

Paste these into any project's `CLAUDE.md` as a baseline. They work standalone or alongside the plugin snippets below.

```markdown
## Agent Guidelines

### Read Before Acting
- NEVER speculate about code you haven't read — always open and read files before answering questions or making claims
- When the user references a file, read it first — give grounded, hallucination-free answers
- Before modifying code, read the surrounding context to understand patterns and conventions

### Plan Before Building
- Think through the problem before writing code
- For non-trivial features or changes touching more than 2-3 files, use `/specsmith-new` to create a proper spec before implementing
- Explain your approach at a high level before diving into details

### Keep It Simple
- Make every change as small and focused as possible
- Prefer editing existing files over creating new ones
- Don't over-engineer — no extra abstractions, no speculative features, no unnecessary refactoring
- If three lines of straightforward code work, don't create a helper function

### Explain As You Go
- Give a brief high-level explanation of each change you make
- After completing a task, summarize what was changed and why

### Verify Your Work
- After making changes, check that nothing is broken
- Run tests if they exist
- If you introduced a bug, fix it before moving on
```

---

## CLAUDE.md Snippets

Each plugin includes a ready-to-paste CLAUDE.md snippet for target projects. Add these to your project's `CLAUDE.md` (after the general guidelines above) to enforce plugin usage.

**Snippet locations:**
- Specsmith: managed via the specsmith-management awareness skill (auto-detects `.specsmiths/`)
- Grimoire: `grimoire/skills/grimoire-awareness/assets/CLAUDE-snippet.md`
- Runebook: `runebook/skills/runebook-awareness/assets/CLAUDE-snippet.md`
- Chisel: `chisel/skills/chisel-awareness/assets/CLAUDE-snippet.md`
- UISpec Backend: `uispec-backend/skills/uispec-backend/assets/CLAUDE-snippet.md`
- UISpec Frontend: `uispec-frontend/skills/uispec-frontend/assets/CLAUDE-snippet.md`

### For Backend Projects

Paste the following snippets into your backend project's `CLAUDE.md`:

<details>
<summary><strong>Runebook snippet</strong> (click to expand)</summary>

```markdown
## Runebook (MANDATORY)

Living documentation lives in `.runebook/`. **ALWAYS consult before changes, ALWAYS update after changes — automatically, without asking.**

### Before ANY Code Changes

You MUST read relevant runebook entries before modifying code:
1. Identify which components will be affected by your changes
2. Read their entries from `.runebook/<type>/<name>.md`
3. Check if a guide covers this domain — read `guides/*.md` for big-picture context
4. Understand current behavior, dependencies, edge cases, and recent changes

### After ANY Code Changes

You MUST update the runebook after changes — do it automatically, do not ask:
- New endpoint → Create `.runebook/endpoints/<name>.md`
- New job → Create `.runebook/jobs/<name>.md`
- New service → Create `.runebook/services/<name>.md`
- New integration → Create `.runebook/integrations/<name>.md`
- Modified component → Update the existing entry
- Removed component → Flag entry as removed
- Guide affected → Update narrative sections in the relevant guide

Commands: /runebook-init, /runebook-update, /runebook-validate, /runebook-show, /runebook-status
```

See `runebook/skills/runebook-awareness/assets/CLAUDE-snippet.md` for the full version with naming conventions, integration notes, and error handling.

</details>

<details>
<summary><strong>Grimoire snippet</strong> (click to expand)</summary>

```markdown
## Grimoire (MANDATORY)

Research suite with 6 specialized agents. **ALWAYS research before guessing.**

- NEVER guess library APIs or best practices — use `/grimoire <query>` to auto-dispatch to the right agents
- For a quick single-stream lookup, prefix with the stream name: `/grimoire docs <library> <question>`
- ALWAYS research before security-sensitive features (auth, crypto, payments, PII)
- ALWAYS analyze the codebase before major refactors

Commands: /grimoire <query>, /grimoire docs, /grimoire practices, /grimoire codebase, /grimoire history, /grimoire schema, /grimoire contracts
```

See `grimoire/skills/grimoire-awareness/assets/CLAUDE-snippet.md` for the full version.

</details>

<details>
<summary><strong>Chisel snippet</strong> (click to expand)</summary>

```markdown
## Chisel (MANDATORY)

Fast surgical code editor. **NEVER manually rename or move files — always use chisel.**

- `/chisel rename <old> to <new>` — rename + update all references
- `/chisel move <source> to <dest>` — move + update all import paths
- `/chisel replace <old> with <new>` — find-and-replace with scope analysis
- `/chisel edit <description>` — small targeted code change

Git-aware (preserves history), framework-aware (finds tests, styles, stories), any language.
```

See `chisel/skills/chisel-awareness/assets/CLAUDE-snippet.md` for the full version.

</details>

<details>
<summary><strong>UISpec Backend snippet</strong> (click to expand)</summary>

```markdown
## UISpec — Backend (MANDATORY)

UI specs live in `../{{PRJ-frontend}}/uispec/`. **ALWAYS sync after API changes.**

After creating/modifying/removing any endpoint:
1. Update `../{{PRJ-frontend}}/uispec/openapi.yaml`
2. Update `../{{PRJ-frontend}}/uispec/endpoints/<endpoint>.md`
3. Update `../{{PRJ-frontend}}/uispec/components.md` if new UI patterns needed

No exceptions. Frontend depends on this.

Commands: /uispec-init, /uispec-sync, /uispec-validate
```

Replace `{{PRJ-frontend}}` with your frontend project directory name. See `uispec-backend/skills/uispec-backend/assets/CLAUDE-snippet.md` for the full version with ownership rules and conflict handling.

</details>

### For Frontend Projects

Paste the following snippets into your frontend project's `CLAUDE.md`:

<details>
<summary><strong>Runebook snippet</strong> (same as backend — click to expand)</summary>

Same snippet as above. See `runebook/skills/runebook-awareness/assets/CLAUDE-snippet.md`.

</details>

<details>
<summary><strong>Grimoire snippet</strong> (same as backend — click to expand)</summary>

Same snippet as above. See `grimoire/skills/grimoire-awareness/assets/CLAUDE-snippet.md`.

</details>

<details>
<summary><strong>Chisel snippet</strong> (same as backend — click to expand)</summary>

Same snippet as above. See `chisel/skills/chisel-awareness/assets/CLAUDE-snippet.md`.

</details>

<details>
<summary><strong>UISpec Frontend snippet</strong> (click to expand)</summary>

```markdown
## UISpec — Frontend (MANDATORY)

UI specs live in `uispec/`. **ALWAYS read specs before building API-connected UI.**

Before creating or modifying any page/component that calls an API endpoint:
1. Read `uispec/endpoints/<endpoint>.md` for the API contract and UI guidelines
2. Read `uispec/components.md` for reusable patterns
3. Read `uispec/design-system.md` for tokens and styling rules

Do not modify `openapi.yaml` or API sections — those are backend-owned.

Commands: /uispec-detect, /uispec-implement, /uispec-validate-ui
```

See `uispec-frontend/skills/uispec-frontend/assets/CLAUDE-snippet.md` for the full version with ownership rules and post-implementation guidance.

</details>

---

## All Commands Reference

| Plugin | Command | Description |
|--------|---------|-------------|
| specsmith | `/specsmith-new <name>: <brief>` | One-shot: discovery + research + full spec |
| specsmith | `/specsmith-new <name>` | Name only: asks for brief, then discovery + research + spec |
| specsmith | `/specsmith-implement` | Start active spec implementation |
| specsmith | `/specsmith-resume` | Re-read everything, continue active spec |
| specsmith | `/specsmith-switch <name>` | Save progress, switch to different spec |
| specsmith | `/specsmith-edit <name>` | Revise spec mid-flight |
| specsmith | `/specsmith-status` | Dashboard of all specs with phase progress |
| specsmith | `/specsmith-list` | Interactive spec list with picker |
| specsmith | `/specsmith-show [name]` | Display spec without activating |
| specsmith | `/specsmith-archive <name>` | Shelve completed/paused spec |
| specsmith | `/specsmith-unarchive <name>` | Restore archived spec |
| specsmith | `/specsmith-drop <name>` | Delete spec + research + questions |
| grimoire | `/grimoire <query>` | Auto-dispatch research (picks relevant agents) |
| grimoire | `/grimoire docs <lib> <query>` | Library docs only |
| grimoire | `/grimoire practices <topic>` | Best practices only |
| grimoire | `/grimoire codebase <area>` | Codebase analysis only |
| grimoire | `/grimoire history <area>` | Git history only |
| grimoire | `/grimoire schema <area>` | DB schema only |
| grimoire | `/grimoire contracts <service>` | External API docs only |
| runebook | `/runebook-init` | Scan codebase, create .runebook/ with entries + guides |
| runebook | `/runebook-update [name]` | Update entry or auto-detect from changes |
| runebook | `/runebook-validate` | Check staleness, coverage, broken refs, guide freshness |
| runebook | `/runebook-show [name]` | Display entry or master index |
| runebook | `/runebook-status` | Dashboard with coverage, staleness, and guide health |
| chisel | `/chisel` | Show available operations |
| chisel | `/chisel rename <old> to <new>` | Rename + update all references |
| chisel | `/chisel move <src> to <dest>` | Move + update all import paths |
| chisel | `/chisel replace <old> with <new>` | Find-and-replace with scope analysis |
| chisel | `/chisel edit <description>` | Small targeted code change |
| uispec-backend | `/uispec-init` | Initialize uispec/ from backend endpoints |
| uispec-backend | `/uispec-sync` | Sync after backend API changes |
| uispec-backend | `/uispec-validate` | Check spec consistency and coverage |
| uispec-frontend | `/uispec-detect` | Find gaps between specs and UI |
| uispec-frontend | `/uispec-implement <endpoint>` | Build/update UI from endpoint spec |
| uispec-frontend | `/uispec-implement <endpoint>: <context>` | Build/update UI with implementation guidance |
| uispec-frontend | `/uispec-validate-ui` | Validate UI follows spec (PASS/FAIL) |

---

## Architecture

### Thin-Command-Over-Thick-Skill Pattern

Each plugin follows a consistent architecture:

- **Commands** (`commands/<name>.md`) — thin wrappers (~15 lines) that declare allowed tools and invoke a skill action
- **Skills** (`skills/<name>/SKILL.md`) — thick implementation containing all logic, state management, and workflows
- **Agents** (`agents/<name>.md`) — specialized subagents spawned via the Task tool for focused work
- **Awareness skills** (`skills/<name>-awareness/SKILL.md`) — proactive triggers that detect when a plugin should activate

### State Management

All state is plain markdown files designed to be committed to git:

- `.specsmiths/<name>.md` — spec with YAML frontmatter (status, phase, steps)
- `.specsmiths/<name>.research.md` — research findings from subagents
- `.specsmiths/<name>.questions.md` — discovery Q&A log
- `.runebook/index.md` — master index with links, tags, dates
- `.runebook/<type>/<name>.md` — entry with YAML frontmatter (endpoints, jobs, flows, services, integrations, pages, components, hooks)
- `.runebook/guides/<name>.md` — narrative system guides organized by feature domain
- `uispec/` — UI specifications (created in frontend projects)
