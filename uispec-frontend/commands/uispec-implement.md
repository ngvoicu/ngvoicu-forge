---
description: "Build or update UI for a specific endpoint based on its uispec. Usage: /uispec-implement <endpoint> or /uispec-implement <endpoint>: <context>"
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, LS
---

# UISpec Implement

Build or update UI for endpoint: `{{ARGS}}`

Invoke the uispec-frontend skill, action: **implement**

The args are: `{{ARGS}}`

**Default: endpoint name only.** If the args contain a colon (e.g. `/uispec-implement create-user: Added email verification — show inline validation`), split on the first colon — everything before it is the endpoint name, everything after is implementation context. The context guides implementation decisions: which components to use, how to handle new states, what changed and why.

If the args have no colon, treat the entire arg as the endpoint name and implement from the spec alone.

Examples with context:
```
/uispec-implement create-user: Added email verification — show inline validation and pending state
/uispec-implement login: Session tokens replaced JWT — remove token refresh logic, add cookie auth
/uispec-implement get-products: Added pagination — use infinite scroll, not page numbers
```

Read the endpoint spec, design system, components docs, and Suggested Implementation section, then generate or update the UI implementation following project conventions.
