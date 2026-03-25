---
description: "Use when: debugging backend or frontend issues with architecture-aware root cause analysis"
name: "Debug Issue"
argument-hint: "Error symptoms, affected endpoint/page, reproduction steps, logs"
agent: "agent"
---
Debug the issue end-to-end using this architecture as baseline.

Inputs:
- Issue details: {{arguments}}
- Architecture reference: [ARCHITECTURE](../../ARCHITECTURE.md)
- Conventions: [CONVENTION](../../CONVENTION.md)
- Environment/routes: [ENV_GUIDELINES](../../ENV_GUIDELINES.md)

Debug workflow:
1. Reproduce and isolate layer: frontend, handler, service, repository, auth, or env.
2. Identify likely root causes with evidence.
3. Implement minimal safe fix.
4. Verify behavior with tests or targeted checks.
5. Confirm no regression in auth, routing, and role rules.

Output requirements:
- Root cause summary
- Exact files changed
- Why the fix works
- Verification results
- Remaining risks or follow-up actions
- Business impact, customer-facing symptoms, and prevention steps
