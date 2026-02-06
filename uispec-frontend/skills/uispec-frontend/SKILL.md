---
name: uispec-frontend
description: "Reads UI specifications from uispec/ in the frontend project to guide component development and API integration. Triggers on: 'check uispec', 'what does the spec say', 'build page for', 'update design', 'new component', or automatically before any API-connected UI work."
---

# UISpec Frontend Skill

Reads `uispec/` in the current frontend project to guide component development, page building, and API integration. This is the consumer side of specs written by the `uispec-backend` skill.

## Structure

```
uispec/
├── openapi.yaml         ← API contract (READ ONLY — backend-owned)
├── UISPEC.md            ← Overview/index
├── design-system.md     ← Tokens, colors, typography (editable)
├── components.md        ← Reusable UI patterns (editable)
└── endpoints/
    └── <endpoint>.md    ← Per-endpoint UI specs (UI sections editable)
```

## Ownership Rules

**Can read (backend-owned, do not modify):**
- `openapi.yaml` — the API contract; changes come from the backend only

**Can read and update (frontend-owned sections):**
- `design-system.md` — design tokens, colors, typography, spacing
- `components.md` — reusable UI component patterns and usage guidelines
- UI Guidelines sections within `endpoints/<endpoint>.md` files (Layout, Components, States, Validation, Accessibility)

**Cannot modify:**
- API sections within endpoint files (Method, Path, Auth, Request, Response) — these are backend-owned
- `openapi.yaml` — backend-owned source of truth

## When to Use

Activate this skill when:
- Building a new page or component that connects to an API endpoint
- The user says "check uispec", "what does the spec say", "build page for [endpoint]"
- The user says "update design", "new component", or references design system tokens
- Before any API-connected UI work — read the endpoint spec first to understand the contract
- When implementing error states, loading states, or form validation — the spec defines expected behavior

## Workflow

### Before Building UI for an Endpoint

1. Read `uispec/endpoints/<endpoint>.md` for the full API contract and UI guidelines
2. Read `uispec/components.md` for reusable patterns that apply
3. Read `uispec/design-system.md` for tokens and styling rules
4. Implement according to the spec:
   - Use the correct request/response shapes from the endpoint file
   - Follow the UI Guidelines section for layout and component choices
   - Handle all documented states (loading, success, error, empty)
   - Apply validation rules specified in the spec
   - Use design system tokens, not hardcoded values

### Updating Frontend-Owned Specs

When the frontend team establishes new patterns:

**Updating `design-system.md`:**
- Add new design tokens as they're established
- Document color palette changes, typography scales, spacing systems
- Keep in sync with the actual CSS/theme implementation

**Updating `components.md`:**
- Add new reusable component patterns as they're built
- Document component props, variants, and usage guidelines
- Reference design system tokens

**Updating UI sections in endpoint files:**
- After building a page, update the UI Guidelines section with actual implementation details
- Document which components were used, specific accessibility attributes added
- Add notes about state management approach

## Rules

1. Always read the endpoint spec before building API-connected UI
2. Never modify `openapi.yaml` — if the API contract is wrong, flag it for the backend team
3. Never modify API sections (Method, Path, Auth, Request, Response) in endpoint files
4. When the spec doesn't cover something, make a reasonable choice and document it in the UI Guidelines section
5. If an endpoint file doesn't exist for an API you need, ask: "No uispec found for this endpoint. Should I request the backend team to create one?"

## CLAUDE.md Snippet

Add this to the frontend project's CLAUDE.md to enforce spec-first UI development. See `assets/CLAUDE-snippet.md` for the full snippet.

---

## Action: detect

Scan the frontend codebase to find gaps between UI specs and implementation. Run all 5 checks and report findings.

### Prerequisites

Verify `uispec/` exists. If not: "No uispec directory found. Ask the backend team to run `/uispec-init`."

### Check 1: Unimplemented specs

1. List all files in `uispec/endpoints/*.md`
2. For each endpoint spec, extract the API path (Method + Path from the spec)
3. Search the codebase for components/pages that call this API path (look for fetch, axios, useSWR, useQuery, useMutation, $http, HttpClient, or similar patterns)
4. Report specs where no UI implementation references the endpoint's API path

### Check 2: Undocumented API calls

1. Scan the codebase for API call patterns: fetch(), axios, useSWR, useQuery, useMutation, $http, HttpClient, api., request(
2. Extract the API paths/URLs from these calls
3. Compare against the list of endpoint specs in `uispec/endpoints/`
4. Report API calls that have no corresponding endpoint spec

### Check 3: Shape mismatches

1. For each endpoint spec that has a corresponding UI implementation:
2. Read the spec's Request and Response sections to identify expected fields
3. Search the implementation for TypeScript interfaces, type definitions, or data access patterns (e.g., `response.data.fieldName`)
4. Compare fields used in code against fields documented in the spec
5. Report field-level differences: missing fields, extra fields, type mismatches

### Check 4: Design system violations

1. Read `uispec/design-system.md` to understand available tokens
2. Scan component files for hardcoded values:
   - Colors: hex codes (#xxx, #xxxxxx), rgb(), rgba(), hsl(), hsla()
   - Font sizes: px/rem/em values for font-size
   - Spacing: hardcoded px/rem margins and paddings
3. For each hardcoded value, check if a matching design token exists
4. Report violations with the file, line, hardcoded value, and suggested token replacement

### Check 5: Missing state handling

1. For each endpoint spec, read the States section (loading, error, empty, success)
2. Find the corresponding UI implementation
3. Check if the component handles each documented state:
   - Loading: look for loading spinners, skeleton screens, isLoading checks
   - Error: look for error boundaries, error messages, catch blocks
   - Empty: look for empty state UI, "no results" messages
4. Report specs where documented states are not handled in the UI

### Output Format

```
UISpec Detection Report
========================
Unimplemented specs:      N endpoints
Undocumented API calls:   N calls
Shape mismatches:         N issues
Design system violations: N found
Missing state handling:   N gaps

Details:
  [spec]  endpoints/<name> — no UI implementation found
  [api]   GET /api/path — no endpoint spec exists
  [shape] endpoints/<name> — response missing `fieldName` field
  [token] src/components/File.tsx:45 — #3B82F6 → use primary-500
  [state] endpoints/<name> — empty state not handled

Suggested fixes:
  - Run /uispec-implement <endpoint> for unimplemented specs
  - Request backend to create specs for undocumented API calls
  - Update TypeScript interfaces to match spec shapes
  - Replace hardcoded values with design system tokens
  - Add missing state handling to components
```

---

## Action: implement

Build or update UI for a specific endpoint based on its uispec. The endpoint name is provided as `{{ARGS}}`.

### Step 1: Validate Input

If `{{ARGS}}` is empty, error: "Please provide an endpoint name. Usage: `/uispec-implement <endpoint>`"

### Step 2: Read Spec Files

1. Read `uispec/endpoints/<endpoint>.md` — the full endpoint spec. If not found: "No spec found for `<endpoint>`. Check available specs in `uispec/endpoints/`."
2. Read `uispec/design-system.md` — design tokens and styling rules
3. Read `uispec/components.md` — reusable UI component patterns

### Step 3: Analyze Existing Implementation

1. Search the codebase for existing components/pages that reference this endpoint's API path
2. If found, read the existing implementation to understand current state
3. Determine if this is a new build or an update to existing code

### Step 4: Determine What to Build

Based on the spec, plan the implementation:
- **Page/component structure**: layout, sections, child components
- **States**: loading, success, error, empty — as documented in the spec's States section
- **Form validation**: required fields, format checks, error display — per the spec's Validation section
- **Accessibility**: ARIA attributes, labels, keyboard navigation — per the spec's Accessibility section
- **Design tokens**: use tokens from `design-system.md`, never hardcoded values
- **Reusable components**: reference patterns from `components.md`

### Step 5: Auto-detect Framework and Conventions

Before generating code, detect the project's framework and conventions:
- **Framework**: React (JSX/TSX), Vue (.vue), Angular, Svelte, etc.
- **Styling**: CSS modules, Tailwind, styled-components, SCSS, CSS-in-JS
- **State management**: React Query, SWR, Redux, Vuex, Pinia, etc.
- **File structure**: how existing pages/components are organized
- **Naming conventions**: PascalCase, kebab-case, file naming patterns

Follow whatever conventions the project uses. Do not impose different patterns.

### Step 6: Generate or Update Code

- If new: create files following the project's file structure and naming conventions
- If update: modify existing files, preserving human-written code and adding missing pieces
- Use the correct request/response shapes from the endpoint spec
- Handle all documented states
- Apply validation rules from the spec
- Use design system tokens, not hardcoded values

### Step 7: Update Spec and Report

1. Update the endpoint's UI Guidelines section with implementation details:
   - Which components were used
   - Accessibility attributes added
   - State management approach
   - Any deviations from the original spec (and why)
2. Append a changelog entry to the endpoint spec: `- [date] UI implemented/updated by uispec-frontend`

**Never modify API sections** (Method, Path, Auth, Request, Response) — these are backend-owned.

### Report

```
UISpec Implement Report
========================
Endpoint:    <endpoint-name>
Action:      Created / Updated

Files created:
  - src/pages/EndpointPage.tsx
  - src/components/EndpointForm.tsx

Files updated:
  - uispec/endpoints/<endpoint>.md (UI Guidelines + changelog)

Spec coverage:
  States:      loading, error, empty, success  ✓
  Validation:  3/3 rules implemented            ✓
  Accessibility: aria-labels, keyboard nav      ✓
  Design tokens: all values use tokens          ✓
```

---

## Action: validate-ui

Validate that existing UI code follows uispec with PASS/FAIL checks. Run all 6 checks across all endpoint implementations.

### Prerequisites

Verify `uispec/` exists. If not: "No uispec directory found. Ask the backend team to run `/uispec-init`."

List all endpoint specs in `uispec/endpoints/`. For each spec that has a corresponding UI implementation, run the following 6 checks.

### Check 1: API contract conformance

- Verify the component uses the correct HTTP method (GET, POST, PUT, DELETE) per spec
- Verify the component calls the correct API path per spec
- Verify authentication headers/tokens are sent per spec's Auth section
- PASS: method, path, and auth all match. FAIL: any mismatch.

### Check 2: Request/response shapes

- Compare TypeScript interfaces or data structures in code against the spec's Request and Response sections
- Check that all required fields are present
- Check that field names match exactly (no casing differences)
- PASS: all fields present and correctly named. FAIL: missing or misnamed fields.

### Check 3: State handling completeness

- Read the spec's States section for documented states (loading, error, empty, success)
- Verify the component renders UI for each documented state
- PASS: all documented states are handled. FAIL: any state is missing.

### Check 4: Design system compliance

- Scan the component files for hardcoded colors, font sizes, and spacing
- Cross-reference against `uispec/design-system.md` tokens
- PASS: all values use design tokens. FAIL: hardcoded values found where tokens exist.

### Check 5: Accessibility compliance

- Read the spec's Accessibility section for required ARIA attributes, labels, keyboard navigation
- Verify the component includes the specified accessibility features
- PASS: all spec'd accessibility requirements are met. FAIL: any missing.
- N/A: spec has no accessibility section for this endpoint.

### Check 6: Form validation compliance

- Read the spec's Validation section for required fields, format checks, error messages
- Verify the component implements the specified validation rules
- PASS: all validation rules implemented. FAIL: any rule missing.
- N/A: endpoint has no form/validation requirements.

### Output Format

```
UISpec UI Validation Report
============================
                              PASS    FAIL    N/A
API contract conformance        N       N       N
Request/response shapes         N       N       N
State handling                  N       N       N
Design system compliance        N       N       N
Accessibility                   N       N       N
Form validation                 N       N       N

FAILURES:
  [API]    endpoints/<name> — uses GET but spec says POST
  [shape]  endpoints/<name> — response missing `fieldName`
  [state]  endpoints/<name> — empty state documented but not handled
  [token]  src/components/File.tsx:23 — #3B82F6 → colors.primary.500
  [a11y]   endpoints/<name> — missing aria-describedby on password field
  [form]   endpoints/<name> — missing email format validation

Suggested fixes:
  - Fix API method/path to match spec
  - Update interfaces to include missing fields
  - Add missing state handling components
  - Replace hardcoded values with design tokens
  - Add required ARIA attributes
  - Implement missing validation rules
```
