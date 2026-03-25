---
description: "Use when: creating a new Go API endpoint following handler-service-repository architecture"
name: "Create Endpoint"
argument-hint: "Service name, endpoint path, method, request/response fields, auth role"
agent: "agent"
---
Create a new Go endpoint that follows project architecture and conventions.

Inputs:
- Feature details: {{arguments}}
- Architecture reference: [ARCHITECTURE](../../ARCHITECTURE.md)
- Conventions reference: [CONVENTION](../../CONVENTION.md)
- Route/env reference: [ENV_GUIDELINES](../../ENV_GUIDELINES.md)

Implementation requirements:
1. Keep Chi router only and route under /api/v1.
2. Add/modify all required layers:
   - handler (validation + transport mapping)
   - service (business logic)
   - repository (gorm persistence if needed)
3. Enforce auth and role checks (student/teacher/admin) where needed.
4. Return standardized HTTP status and error payloads.
5. If delete-like behavior is requested, implement soft delete.
6. Add or update DTOs and model mapping carefully.
7. Add tests:
   - handler test (success + validation + auth failure)
   - service test for business rules

Expected output:
- Summary of files changed
- Key design choices
- Follow-up TODOs if any assumptions were required
- Business rule coverage and acceptance criteria for the endpoint
