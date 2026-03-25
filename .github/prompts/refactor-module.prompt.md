---
description: "Use when: refactoring a Go or Svelte module without changing behavior"
name: "Refactor Module"
argument-hint: "Module path, refactor goals, constraints, performance/test expectations"
agent: "agent"
---
Refactor the target module while preserving behavior and architecture contracts.

Inputs:
- Refactor request: {{arguments}}
- Architecture reference: [ARCHITECTURE](../../ARCHITECTURE.md)
- Conventions: [CONVENTION](../../CONVENTION.md)

Rules:
1. Preserve external behavior and API contracts.
2. Keep or improve layering boundaries.
3. Remove duplication and improve readability.
4. Keep auth/rbac and route semantics intact.
5. Avoid unrelated formatting churn.
6. Update/add tests to lock behavior before and after.

Expected output:
- What was refactored and why
- Evidence behavior stayed equivalent
- Risks and any deferred cleanup tasks
- Proof that business behavior and permissions remained unchanged
