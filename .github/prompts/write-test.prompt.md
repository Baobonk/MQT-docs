---
description: "Use when: writing tests for Go handlers/services/repositories or Svelte pages/components"
name: "Write Test"
argument-hint: "Target file/symbol, test level, scenarios to cover"
agent: "agent"
---
Write high-value tests aligned with architecture and conventions.

Inputs:
- Test target: {{arguments}}
- Architecture reference: [ARCHITECTURE](../../ARCHITECTURE.md)
- Conventions reference: [CONVENTION](../../CONVENTION.md)

Requirements:
1. Identify the right test level:
   - unit (fast, isolated)
   - integration (cross-layer behavior)
2. Prioritize meaningful scenarios:
   - happy path
   - validation failures
   - auth/rbac failures
   - persistence failures/timeouts
   - edge cases and regressions
3. Follow existing patterns and naming style.
4. Keep tests deterministic and readable.
5. Include assertions for status codes, payload shape, and side effects.

Output:
- New/updated test files
- Scenario checklist covered
- Gaps that still require integration/e2e coverage
- Mapping of test scenarios to business rules
