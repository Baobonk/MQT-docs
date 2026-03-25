---
description: "Use when: implementing or updating service-to-service or frontend-to-service API calls"
name: "Service Call"
argument-hint: "Caller, callee, endpoint, payload, auth requirement, error handling"
agent: "agent"
---
Implement a service call consistent with this microservice architecture.

Inputs:
- Call details: {{arguments}}
- Architecture reference: [ARCHITECTURE](../../ARCHITECTURE.md)
- Environment/routes: [ENV_GUIDELINES](../../ENV_GUIDELINES.md)

Requirements:
1. Resolve base URL and path from environment config and /api/v1 ownership.
2. Include JWT bearer token for protected APIs.
3. Add timeout, retry (when safe), and structured error mapping.
4. Validate request/response contracts and map to DTOs.
5. Avoid leaking transport details into business logic.
6. Add observability:
   - request id propagation when available
   - useful logs for failures
7. Add tests for:
   - happy path
   - timeout/network failure
   - unauthorized/forbidden responses

Output format:
- Files changed
- Request/response contract summary
- Failure modes handled
- Business flow impact and rollback considerations
