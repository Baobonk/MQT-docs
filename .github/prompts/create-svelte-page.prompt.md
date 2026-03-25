---
description: "Use when: creating a new Svelte page connected to Go backend APIs in this architecture"
name: "Create Svelte Page"
argument-hint: "Page name, route, required data, role visibility, API endpoints"
agent: "agent"
---
Create a new Svelte page and wire it to backend services using project standards.

Inputs:
- Page requirements: {{arguments}}
- Architecture reference: [ARCHITECTURE](../../ARCHITECTURE.md)
- Route/env reference: [ENV_GUIDELINES](../../ENV_GUIDELINES.md)

Requirements:
1. Add route and page structure aligned with current frontend setup.
2. Use centralized API client/config from environment variables.
3. Call backend endpoints under /api/v1 with proper JWT header if protected.
4. Implement role-aware behavior for student/teacher/admin where needed.
5. Provide complete UX states:
   - loading
   - empty
   - error
   - success
6. Keep business logic out of markup where possible (use helpers/stores).
7. Add page-level tests for core interactions and authorization behavior.

Output:
- Files created/updated
- New route details
- API endpoints consumed
- Business journey acceptance criteria satisfied by the page
