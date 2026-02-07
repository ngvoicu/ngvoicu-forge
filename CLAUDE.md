# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **ngvoicu-forge-claude-code**, a Claude Code plugin marketplace containing six plugins for spec-driven development, deep research, living documentation, surgical code editing, and UI specification workflows. All plugins are markdown-based (commands, skills, assets) with no build system, package manager, or test framework.

## Architecture

### Plugin Structure

Each plugin follows a **thin-command-over-thick-skill** pattern:

- `commands/<name>.md` — thin wrappers (~14 lines) that declare allowed tools and invoke a skill action
- `skills/<name>/SKILL.md` — thick implementation containing all logic
- `agents/<name>.md` — specialized subagents spawned via the Task tool for focused work

### Six Plugins

**specsmith** — Research-driven implementation specs with discovery interviews, parallel research subagents, and phased execution. Supports one-shot spec creation (`/specsmith-new name: brief`), a quality gate that rejects vague briefs before analysis, and lightweight research mode for simple changes. State lives in `.specsmiths/` as plain markdown committed to git.

**grimoire** — Research suite with 6 specialized agents that auto-dispatches based on your query. Just `/grimoire <query>` — it picks the right agents (docs, best practices, codebase, git history, DB schema, API contracts) automatically. Integrates with specsmith Phase 2 for enhanced research.

**runebook** — Living documentation that tracks how an application works. Maintains structured docs for endpoints, jobs, flows, services, integrations, pages, components, and hooks — covering both backend and frontend. Also generates narrative **system guides** organized by feature domain (authentication, payments, onboarding) that explain how things work end-to-end. Guides are auto-generated from component relationships and kept in sync with code changes, while preserving human-written sections. Always in sync with code changes. State lives in `.runebook/` as plain markdown committed to git.

**chisel** — Fast surgical code editor (haiku model) for renaming, moving, replacing, and small targeted edits. Git-aware, framework-aware, language-agnostic. Automatically updates all references and imports.

**uispec-backend** — Syncs `openapi.yaml` and endpoint specs in `../{{PRJ-frontend}}/uispec/` after backend API changes. Also generates a Suggested Implementation section in each endpoint file with component recommendations, key interactions, and things to watch out for (rate limits, file uploads, nested objects). Backend-owned: `openapi.yaml`, API sections, Suggested Implementation.

**uispec-frontend** — Frontend commands for detecting spec gaps, building UI from specs, and validating UI conformance. Supports optional context via colon syntax (`/uispec-implement endpoint: context`) to guide implementation decisions when updating existing endpoints. Frontend-owned: `design-system.md`, `components.md`, UI Guidelines sections of endpoint files.

### Key Directories

- `.claude-plugin/marketplace.json` — plugin registry (name, source path, description for each plugin)
- `specsmith/.claude-plugin/plugin.json` — specsmith manifest
- `grimoire/.claude-plugin/plugin.json` — grimoire manifest
- `runebook/.claude-plugin/plugin.json` — runebook manifest
- `chisel/.claude-plugin/plugin.json` — chisel manifest
- `uispec-backend/.claude-plugin/plugin.json` — uispec-backend manifest
- `uispec-frontend/.claude-plugin/plugin.json` — uispec-frontend manifest
- `.specsmiths/` — spec state storage (created per-project by specsmith, not in this repo)
- `.runebook/` — runebook state storage (created per-project by runebook, not in this repo)
- `uispec/` — UI specification directory (created in frontend projects, not in this repo)

### State Management

All state is plain markdown files designed to be committed to git:

- `.specsmiths/<name>.md` — spec with YAML frontmatter (`status`, `current_phase`, `total_phases`)
- `.specsmiths/<name>.research.md` — research findings from subagents
- `.specsmiths/<name>.questions.md` — discovery Q&A log
- `.specsmiths/active.json` — tracks active spec (`{"active": "<name>", "last_switched": "<ISO date>"}`)
- `.runebook/index.md` — master index with links, tags, dates
- `.runebook/<type>/<name>.md` — entry with YAML frontmatter (`type`, `updated`, `source_files`, `related`, `tags`). Types: `endpoints/`, `jobs/`, `flows/`, `services/`, `integrations/`, `pages/`, `components/`, `hooks/`
- `.runebook/guides/<name>.md` — narrative system guides with YAML frontmatter (`type: guide`, `domain`, `covers`, `tags`). Organized by feature domain, not component type.

### Grimoire Agent Streams

| Stream | Agent | Purpose |
|--------|-------|---------|
| `docs` | grimoire-docs | Library documentation (Context7 primary, web fallback) |
| `practices` | grimoire-practices | Best practices, pitfalls, security considerations |
| `codebase` | grimoire-codebase | Architecture, patterns, files affected |
| `history` | grimoire-history | Git log, TODOs, ADRs, prior attempts |
| `schema` | grimoire-schema | DB schema, migrations, indexes |
| `contracts` | grimoire-contracts | External API docs, rate limits, auth |

### Specsmith Three-Phase Workflow

1. **Discovery** — one-shot brief via `name: brief` syntax (default) or interactive. Brief quality gate rejects vague input before analysis. Interview rounds only when needed — if the brief is comprehensive, skips straight to research.
2. **Research** — adaptive parallel subagents (up to 6 with grimoire, up to 5 with inline fallback); uses grimoire agents when installed, falls back to inline subagents. Lightweight mode (codebase-only) available for simple changes when user requests it.
3. **Spec creation** — full specification with requirements, architecture, phased steps (What/Where/How/Verify), edge case table, risks, testing strategy, rollback plan

### UISpec Ownership Rules

Backend-owned (read-only for frontend): `openapi.yaml`, API sections in endpoint files (Method, Path, Auth, Request, Response), Suggested Implementation section

Frontend-owned (read-only for backend): `design-system.md`, `components.md`, UI Guidelines sections in endpoint files

Suggested Implementation is backend-generated but frontend can override — it provides component suggestions, key interactions, and watchouts based on endpoint patterns

### Specsmith + UISpec Full-Stack Workflow

**New features:** specsmith creates the spec → backend implements API endpoints → `/uispec-sync` pushes API contracts to frontend → `/uispec-implement <endpoint>` builds UI from spec → `/uispec-validate-ui` confirms alignment. Specsmith reminds you to sync at step and phase boundaries when API endpoints change.

**Updating existing endpoints:** backend changes → `/uispec-sync` updates spec files → `/uispec-detect` shows gaps → `/uispec-implement <endpoint>: <what changed and how to handle it>` updates UI with context. The colon syntax provides implementation guidance (e.g., `create-user: Added email verification — show inline validation`) so uispec knows the intent behind spec changes, not just the shape diff.

## Commands

### Specsmith (11 commands)
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

### Grimoire (1 command, auto-dispatches)
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

### Runebook (5 commands)
```
/runebook-init              # scan codebase, create .runebook/ with entries + guides
/runebook-update [name]     # update specific entry or auto-detect from changes
/runebook-validate          # check staleness, coverage, broken refs, guide freshness
/runebook-show [name]       # display entry or master index
/runebook-status            # dashboard with coverage, staleness, and guide health
```

### Chisel (1 command, 4 operations)
```
/chisel                                # show available operations
/chisel rename <old> to <new>          # rename + update all references
/chisel move <source> to <dest>        # move + update all import paths
/chisel replace <old> with <new>       # find-and-replace with scope analysis
/chisel edit <description>             # small targeted code change
```

### UISpec Backend (3 commands)
```
/uispec-init      # initialize uispec/ from existing backend endpoints
/uispec-sync      # sync after backend API changes
/uispec-validate  # check spec consistency and coverage
```

### UISpec Frontend (3 commands)
```
/uispec-detect                          # find gaps between specs and UI
/uispec-implement <endpoint>            # build/update UI from endpoint spec
/uispec-implement <endpoint>: <context> # build/update with implementation guidance
/uispec-validate-ui                     # validate UI follows spec (PASS/FAIL)
```

## Critical Rules

### Specsmith
- **Never use plan mode (EnterPlanMode) for specsmith commands** — specsmith IS the planning process. It handles its own discovery, research, and spec creation. If plan mode is active when a `/specsmith-*` command is invoked, exit plan mode first before executing the command.
- **One-shot is the default** — `name: brief` syntax lets users provide everything in one command. Only ask for the brief separately if name-only was provided.
- **Brief quality gate** — reject vague briefs (no specific behavior, no trigger/problem, single vague phrase) before spending analysis time. Ask user to expand.
- Discovery and research phases are **mandatory** — never skip them. Lightweight research (codebase-only) is allowed only when the user explicitly requests it for simple changes (1-2 files, existing dependencies).
- Specs are the **source of truth** — never rely on conversation memory
- Always **re-read spec + research + questions on resume** to restore full context
- Every implementation step must be **implementable without follow-up questions**
- Every step must have **concrete verification criteria**
- Edge cases are **first-class** — tracked in a table, assigned to specific implementation steps
- **Completed steps are sacred** — never silently modify checked-off work

### Grimoire
- **Never guess — research first** — use `/grimoire <query>` to auto-dispatch to the right agents. Grimoire picks which streams to use based on your query.
- **Always research best practices** before implementing security-sensitive features (auth, crypto, payments, PII)
- **Always analyze the codebase** before major refactors to understand existing patterns
- For a quick single-stream lookup, prefix with the stream name: `/grimoire docs <lib> <query>`

### Runebook
- **Runebook is assertive** — always read entries and guides before code work, always update after changes without asking
- **Human context is sacred** — never overwrite notes, explanations, or annotations humans added
- **Changelog is append-only** — never remove or modify existing changelog entries
- **Cross-references are bidirectional** — if entry A references B, entry B must reference A
- **Guides are narrative** — they explain how things work in plain language, not as component lists. Auto-derivable sections (How It Works, Architecture, Where Things Live) update from code; human sections (Key Concepts, Gotchas, Common Tasks) are preserved always.

### Chisel
- **Never manually rename or move files** — always use `/chisel` to ensure all references are updated
- **Never use sed/awk for find-and-replace** — use `/chisel replace` which handles word boundaries and verification
- After chisel operations, **update runebook entries** referencing affected files

### UISpec
- Always **update `openapi.yaml` first** when syncing uispec — it's the API source of truth
- **Respect ownership boundaries** — frontend doesn't modify API sections, backend doesn't modify UI sections
- **Suggested Implementation** — backend generates component suggestions, key interactions, and watchouts during sync. Frontend can override in UI Guidelines.
- **Context via colon syntax** — `/uispec-implement endpoint: context` provides implementation guidance for updates. The context explains why things changed and how the UI should handle it. Without context, uispec implements spec changes mechanically from the shape diff alone.

### Cross-Plugin Workflows
- **specsmith + grimoire**: Phase 2 uses grimoire agents for research when available
- **specsmith + runebook**: Read entries during implementation, update after each step
- **specsmith + uispec**: After implementing spec steps that change API endpoints, run `/uispec-sync` to update the API contract. Specsmith reminds you at step and phase boundaries.
- **chisel + runebook**: After rename/move, update runebook `source_files` references
- **grimoire + chisel**: Research scope with `/grimoire codebase` before large renames
- **chisel + uispec**: After renaming endpoints or API handlers, update corresponding uispec endpoint spec files
- **uispec + runebook**: Keep endpoint documentation consistent across both

## Installation

```bash
/plugin marketplace add https://github.com/ngvoicu/ngvoicu-forge-claude-code.git
/plugin install specsmith@ngvoicu-forge-claude-code
/plugin install grimoire@ngvoicu-forge-claude-code
/plugin install runebook@ngvoicu-forge-claude-code
/plugin install chisel@ngvoicu-forge-claude-code
/plugin install uispec-backend@ngvoicu-forge-claude-code
/plugin install uispec-frontend@ngvoicu-forge-claude-code
```

## CLAUDE.md Snippets

Each plugin includes a ready-to-paste CLAUDE.md snippet for target projects:
- Grimoire: `grimoire/skills/grimoire-awareness/assets/CLAUDE-snippet.md`
- Runebook: `runebook/skills/runebook-awareness/assets/CLAUDE-snippet.md`
- Chisel: `chisel/skills/chisel-awareness/assets/CLAUDE-snippet.md`
- UISpec Backend: `uispec-backend/skills/uispec-backend/assets/CLAUDE-snippet.md`
- UISpec Frontend: `uispec-frontend/skills/uispec-frontend/assets/CLAUDE-snippet.md`
