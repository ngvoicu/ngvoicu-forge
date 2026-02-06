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

**specsmith** — Research-driven implementation specs with discovery interviews, parallel research subagents, and phased execution. State lives in `.specsmiths/` as plain markdown committed to git.

**grimoire** — Research suite with 6 specialized agents: docs (Context7 + web fallback), best practices, codebase analysis, git history, DB schema, and API contracts. Integrates with specsmith Phase 2 for enhanced research.

**runebook** — Living documentation that tracks how an application works. Maintains structured docs for endpoints, jobs, flows, services, integrations, pages, components, and hooks — covering both backend and frontend. Always in sync with code changes. State lives in `.runebook/` as plain markdown committed to git.

**chisel** — Fast surgical code editor (haiku model) for renaming, moving, replacing, and small targeted edits. Git-aware, framework-aware, language-agnostic. Automatically updates all references and imports.

**uispec-backend** — Syncs `openapi.yaml` and endpoint specs in `../{{PRJ-frontend}}/uispec/` after backend API changes. Backend-owned: `openapi.yaml`, API sections of endpoint files.

**uispec-frontend** — Frontend commands for detecting spec gaps, building UI from specs, and validating UI conformance. Frontend-owned: `design-system.md`, `components.md`, UI Guidelines sections of endpoint files.

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

1. **Discovery** — interview rounds until requirements, edge cases, and scope are deeply understood
2. **Research** — 1-6 adaptive parallel subagents; uses grimoire agents when installed, falls back to inline subagents
3. **Spec creation** — full specification with requirements, architecture, phased steps (What/Where/How/Verify), edge case table, risks, testing strategy, rollback plan

### UISpec Ownership Rules

Backend-owned (read-only for frontend): `openapi.yaml`, API sections in endpoint files (Method, Path, Auth, Request, Response)

Frontend-owned (read-only for backend): `design-system.md`, `components.md`, UI Guidelines sections in endpoint files

## Commands

### Specsmith (11 commands)
```
/specsmith-new <name>       # discovery -> research -> full spec
/specsmith-implement        # start active spec implementation
/specsmith-resume           # re-read everything, continue active spec
/specsmith-switch <name>    # save progress, switch active spec
/specsmith-edit <name>      # revise spec mid-flight
/specsmith-status           # dashboard of all specs with phase progress
/specsmith-list             # interactive list with picker
/specsmith-show [name]      # display spec without activating
/specsmith-archive <name>   # shelve completed/paused spec
/specsmith-unarchive <name> # restore archived spec
/specsmith-drop <name>      # delete spec + research + questions
```

### Grimoire (1 command)
```
/grimoire                          # show available streams
/grimoire docs <lib> <query>       # library documentation lookup
/grimoire practices <topic>        # best practices & pitfalls
/grimoire codebase <area>          # codebase analysis
/grimoire history <area>           # git history & prior work
/grimoire schema <area>            # DB schema analysis
/grimoire contracts <service>      # external API contracts
/grimoire research <context>       # full parallel research
```

### Runebook (5 commands)
```
/runebook-init              # scan codebase, create .runebook/
/runebook-update [name]     # update specific entry or auto-detect from changes
/runebook-validate          # check staleness, coverage, broken refs
/runebook-show [name]       # display entry or master index
/runebook-status            # dashboard with coverage and staleness
```

### Chisel (1 command)
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
/uispec-detect                # find gaps between specs and UI
/uispec-implement <endpoint>  # build/update UI from endpoint spec
/uispec-validate-ui           # validate UI follows spec (PASS/FAIL)
```

## Critical Rules

### Specsmith
- **Never use plan mode (EnterPlanMode) for specsmith commands** — specsmith IS the planning process. It handles its own discovery, research, and spec creation. If plan mode is active when a `/specsmith-*` command is invoked, exit plan mode first before executing the command.
- Discovery and research phases are **mandatory** — never skip them
- Specs are the **source of truth** — never rely on conversation memory
- Always **re-read spec + research + questions on resume** to restore full context
- Every implementation step must be **implementable without follow-up questions**
- Every step must have **concrete verification criteria**
- Edge cases are **first-class** — tracked in a table, assigned to specific implementation steps
- **Completed steps are sacred** — never silently modify checked-off work

### Grimoire
- **Never guess library APIs** — always use `/grimoire docs` to look up exact signatures when uncertain
- **Always research best practices** before implementing security-sensitive features (auth, crypto, payments, PII)
- **Always analyze the codebase** before major refactors to understand existing patterns

### Runebook
- **Runebook is assertive** — always read entries before code work, always update after changes without asking
- **Human context is sacred** — never overwrite notes, explanations, or annotations humans added
- **Changelog is append-only** — never remove or modify existing changelog entries
- **Cross-references are bidirectional** — if entry A references B, entry B must reference A

### Chisel
- **Never manually rename or move files** — always use `/chisel` to ensure all references are updated
- **Never use sed/awk for find-and-replace** — use `/chisel replace` which handles word boundaries and verification
- After chisel operations, **update runebook entries** referencing affected files

### UISpec
- Always **update `openapi.yaml` first** when syncing uispec — it's the API source of truth
- **Respect ownership boundaries** — frontend doesn't modify API sections, backend doesn't modify UI sections

### Cross-Plugin Workflows
- **specsmith + grimoire**: Phase 2 uses grimoire agents for research when available
- **specsmith + runebook**: Read entries during implementation, update after each step
- **chisel + runebook**: After rename/move, update runebook `source_files` references
- **grimoire + chisel**: Research scope with `/grimoire codebase` before large renames
- **specsmith + uispec**: After implementing spec steps that change API endpoints, run `/uispec-sync` to update the API contract. Specsmith reminds you at step and phase boundaries.
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
