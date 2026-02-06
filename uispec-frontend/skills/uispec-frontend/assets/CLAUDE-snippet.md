## UISpec — Frontend (MANDATORY)

UI specs live in `uispec/`. **ALWAYS read specs before building API-connected UI.**

### When to Use UISpec

| Situation | Action |
|-----------|--------|
| Building a new page/component that calls an API | Read `uispec/endpoints/<endpoint>.md` first |
| Styling API-connected components | Read `uispec/design-system.md` for tokens |
| Looking for reusable UI patterns | Read `uispec/components.md` |
| Implementing form validation | Follow the spec's Validation rules section |
| Handling loading/error/empty states | Follow the spec's States section |
| After building UI for an endpoint | Update the endpoint's UI Guidelines section |

### Before Building UI

Before creating or modifying any page/component that calls an API endpoint:
1. Read `uispec/endpoints/<endpoint>.md` for the API contract and UI guidelines
2. Read `uispec/components.md` for reusable patterns
3. Read `uispec/design-system.md` for tokens and styling rules
4. Follow the spec's States section (loading, success, error, empty)
5. Apply the spec's Validation rules before API calls

### Rules

- **NEVER modify `openapi.yaml`** — this is backend-owned, changes come from backend only
- **NEVER modify API sections** in endpoint files (Method, Path, Auth, Request, Response) — these are backend-owned
- **ALWAYS read the spec first** before building any API-connected component
- **ALWAYS update UI Guidelines** after building — document which components were used, accessibility attributes, state management approach
- **ALWAYS flag API discrepancies** — if the actual API response doesn't match the spec, flag it for the backend team, do not silently adapt

### Ownership Boundaries

| Section | Owner | Frontend Can |
|---------|-------|--------------|
| Method, Path, Auth, Request, Response, Errors | Backend | Read only |
| UI Guidelines, Components, States, Accessibility | Frontend | Read + Write |
| `design-system.md` | Frontend | Read + Write |
| `components.md` | Frontend | Read + Write |
| `openapi.yaml` | Backend | Read only |

### After Building UI

After implementing a page/component, update the endpoint's UI Guidelines section with:
- Which components were actually used
- Accessibility attributes added
- State management approach
- Any deviations from the original spec (and why)

### Commands

```
/uispec-validate  # Check spec consistency and coverage
```

### Integration with Other Plugins

**With runebook:** If `.runebook/` exists, read relevant endpoint and service entries for additional context about how APIs behave, including edge cases and dependencies.

**With specsmith:** When implementing a spec that involves UI work, read relevant uispec entries during implementation to ensure the UI matches the documented API contract.

**With grimoire:** Use `/grimoire docs` to check component library APIs before building. Use `/grimoire practices` for accessibility and UX best practices.

**With chisel:** When renaming components that reference uispec endpoints, use `/chisel rename` to update both component files and uispec references.

### Notes

- If `uispec/` doesn't exist, ask the backend team to run `/uispec-init` — do not create it yourself
- If an endpoint spec doesn't exist for an API you need, stop and flag it for the backend team
- If the API contract doesn't match the actual API response, flag the discrepancy immediately
