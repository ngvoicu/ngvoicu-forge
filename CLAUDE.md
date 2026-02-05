# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **ngvoicu-forge-claude-code**, a Claude Code plugin marketplace containing three plugins for spec-driven development and UI specification workflows. All plugins are markdown-based (commands, skills, assets) with no build system, package manager, or test framework.

## Architecture

### Plugin Structure

Each plugin follows a **thin-command-over-thick-skill** pattern:

- `commands/<name>.md` — thin wrappers (~14 lines) that declare allowed tools and invoke a skill action
- `skills/<name>/SKILL.md` — thick implementation containing all logic

### Three Plugins

**specsmith** — Research-driven implementation specs with discovery interviews, parallel research subagents, and phased execution. State lives in `.specsmiths/` as plain markdown committed to git.

**uispec-backend** — Syncs `openapi.yaml` and endpoint specs in `../{{PRJ-frontend}}/uispec/` after backend API changes. Backend-owned: `openapi.yaml`, API sections of endpoint files.

**uispec-frontend** — Reads specs from `uispec/` to guide component development. Frontend-owned: `design-system.md`, `components.md`, UI Guidelines sections of endpoint files.

### Key Directories

- `.claude-plugin/marketplace.json` — plugin registry (name, source path, description for each plugin)
- `specsmith/.claude-plugin/plugin.json` — specsmith manifest
- `uispec-backend/.claude-plugin/plugin.json` — uispec-backend manifest
- `uispec-frontend/.claude-plugin/plugin.json` — uispec-frontend manifest
- `.specsmiths/` — spec state storage (created per-project by specsmith, not in this repo)
- `uispec/` — UI specification directory (created in frontend projects, not in this repo)

### State Management

All state is plain markdown files designed to be committed to git:

- `.specsmiths/<name>.md` — spec with YAML frontmatter (`status`, `current_phase`, `total_phases`)
- `.specsmiths/<name>.research.md` — research findings from subagents
- `.specsmiths/<name>.questions.md` — discovery Q&A log
- `.specsmiths/active.json` — tracks active spec (`{"active": "<name>", "last_switched": "<ISO date>"}`)

### Specsmith Three-Phase Workflow

1. **Discovery** — interview rounds until requirements, edge cases, and scope are deeply understood
2. **Research** — 1-5 adaptive parallel subagents (codebase, best practices, library docs, git history, DB schema)
3. **Spec creation** — full specification with requirements, architecture, phased steps (What/Where/How/Verify), edge case table, risks, testing strategy, rollback plan

### UISpec Ownership Rules

Backend-owned (read-only for frontend): `openapi.yaml`, API sections in endpoint files (Method, Path, Auth, Request, Response)

Frontend-owned (read-only for backend): `design-system.md`, `components.md`, UI Guidelines sections in endpoint files

## Commands

### Specsmith (11 commands)
```
/specsmith-new <name>       # discovery → research → full spec
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

### UISpec (3 commands)
```
/uispec-init      # initialize uispec/ from existing backend endpoints
/uispec-sync      # sync after backend API changes
/uispec-validate  # check spec consistency and coverage
```

## Critical Rules

- Discovery and research phases are **mandatory** — never skip them
- Specs are the **source of truth** — never rely on conversation memory
- Always **re-read spec + research + questions on resume** to restore full context
- Every implementation step must be **implementable without follow-up questions**
- Every step must have **concrete verification criteria**
- Edge cases are **first-class** — tracked in a table, assigned to specific implementation steps
- **Completed steps are sacred** — never silently modify checked-off work
- Always **update `openapi.yaml` first** when syncing uispec — it's the API source of truth
- **Respect ownership boundaries** — frontend doesn't modify API sections, backend doesn't modify UI sections

## Installation

```bash
/plugin marketplace add https://github.com/ngvoicu/ngvoicu-forge-claude-code.git
/plugin install specsmith@ngvoicu-forge-claude-code
/plugin install uispec@ngvoicu-forge-claude-code
```

## CLAUDE.md Snippets

Each uispec plugin includes a ready-to-paste CLAUDE.md snippet for target projects:
- Backend: `uispec-backend/skills/uispec-backend/assets/CLAUDE-snippet.md`
- Frontend: `uispec-frontend/skills/uispec-frontend/assets/CLAUDE-snippet.md`
