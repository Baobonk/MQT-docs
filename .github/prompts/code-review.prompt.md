---
description: "Use when: performing a code review for Go microservices and Svelte frontend in this architecture"
name: "Code Review"
argument-hint: "Scope, files, and risk focus (security/performance/regression)"
agent: "agent"
---
Review the requested changes against this architecture:
- Frontend: Svelte 4
- Backend: Go microservices with Chi Router + GORM + PostgreSQL
- Auth: Keycloak JWT
- API prefix: /api/v1
- Layering: handler -> service -> repository

Inputs:
- Scope: {{arguments}}
- Architecture reference: [ARCHITECTURE](../../ARCHITECTURE.md)
- Backend conventions: [CONVENTION](../../CONVENTION.md)
- Environment and routes: [ENV_GUIDELINES](../../ENV_GUIDELINES.md)

Output requirements:
1. Findings first, ordered by severity: critical, major, minor.
2. For each finding include:
   - File and line references
   - Why this is a bug/risk/regression
   - Concrete fix suggestion
3. Check architecture consistency:
   - Wrong router/framework usage
   - Broken layer boundaries
   - Missing auth/rbac checks
   - API prefix/routing drift from /api/v1
   - Missing soft delete behavior
4. Check testing coverage gaps for changed logic.
5. If no findings, explicitly state no findings and list residual risks.
6. Add a business impact note for each critical and major finding.
