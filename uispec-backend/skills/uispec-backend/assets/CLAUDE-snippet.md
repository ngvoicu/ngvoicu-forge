## UISpec — Backend (MANDATORY)

UI specs live in `../{{PRJ-frontend}}/uispec/`. **ALWAYS sync after API changes.**

### When to Sync

| Change | Action |
|--------|--------|
| New endpoint created | Run sync: update openapi.yaml + create endpoint spec |
| Request/response shape modified | Run sync: update openapi.yaml + update endpoint spec |
| Auth/permissions changed on endpoint | Run sync: update endpoint spec auth section |
| Endpoint removed or deprecated | Run sync: mark endpoint as deprecated in spec |
| New error response formats added | Run sync: update endpoint spec error section |
| Pagination/filtering/sorting changed | Run sync: update endpoint spec query params |

### How to Sync

After creating/modifying/removing any endpoint:
1. Update `../{{PRJ-frontend}}/uispec/openapi.yaml`
2. Create/update `../{{PRJ-frontend}}/uispec/endpoints/<endpoint>.md`
3. Update `../{{PRJ-frontend}}/uispec/components.md` if new UI patterns needed
4. Update `../{{PRJ-frontend}}/uispec/UISPEC.md` index if endpoints added/removed

No exceptions. Frontend depends on this.

### Rules

- **ALWAYS update openapi.yaml first** — it is the API source of truth
- **NEVER modify frontend-owned sections** — UI Guidelines, Components, Accessibility sections belong to the frontend team
- **ALWAYS preserve manual additions** — if frontend added notes to a spec, merge changes without overwriting
- **ALWAYS add a changelog entry** — every sync appends to the endpoint's changelog with date and what changed

### Ownership Boundaries

| Section | Owner | Backend Can |
|---------|-------|-------------|
| Method, Path, Auth, Request, Response, Errors | Backend | Read + Write |
| UI Guidelines, Components, States, Accessibility | Frontend | Read only |
| Changelog | Shared | Append only |

### First-Time Setup

If `../{{PRJ-frontend}}/uispec/` doesn't exist yet, run `/uispec-init` to initialize the directory structure and generate specs from existing endpoints.

### Commands

```
/uispec-init      # Initialize uispec/ from existing backend endpoints
/uispec-sync      # Sync after backend API changes
/uispec-validate  # Check spec consistency and coverage
```

### Integration with Other Plugins

**With runebook:** If `.runebook/` exists, update runebook endpoint entries after syncing uispec to keep both documentation sources consistent.

**With specsmith:** When implementing a spec that involves API changes, sync uispec after each endpoint-related implementation step.

**With grimoire:** Use `/grimoire contracts` to check external API docs before designing new endpoint contracts.

**With chisel:** When renaming endpoints or API handlers, use `/chisel rename` to update both source files and corresponding uispec spec files.

### Notes

- If `../{{PRJ-frontend}}/` doesn't exist, warn and stop — do not create the frontend directory
- If spec files were manually edited by the frontend team, read existing content first and merge carefully
- After sync, validate that openapi.yaml is valid YAML and all endpoint files are consistent
- Replace `{{PRJ-frontend}}` with your actual frontend project directory name
