# Go Backend Instruction

This document defines the implementation rules for all Go microservices in this project.

## Business Outcome Requirement

Every backend change must preserve or improve at least one business outcome:

- learner activation and progression
- teacher productivity and class outcomes
- revenue integrity and billing correctness
- operational governance and auditability

If a change risks business-rule correctness, stop and redesign before implementation.

## 1) Core Stack and Hard Rules

- Language: Go
- Router: Chi only (`github.com/go-chi/chi/v5`)
- ORM: GORM
- Database: PostgreSQL
- Auth: Keycloak + JWT bearer token
- Shared API prefix: `/api/v1`
- User role enum values must be exactly: `student`, `teacher`, `admin`

Not allowed:
- `http.ServeMux`, `gorilla/mux`, `gin`, `echo`, `fiber`
- Permanent delete for business entities that require archiving

## 2) Service Architecture

Each service must follow this layering:

- Handler layer: transport mapping, request parsing, validation, response mapping
- Service layer: business rules, orchestration, policy enforcement
- Repository layer: database access using GORM

Recommended service layout:

```text
/service-name
  handlers/
  services/
  repositories/
  models/
  middleware/
  routes.go
  main.go
```

## 3) Routing Standard

- All endpoints are exposed under `/api/v1`
- Resource naming is noun-based and consistent with bounded contexts
- Use path params such as `{id}`, `{courseId}`, `{moduleId}`
- Use query params for filtering, sort, pagination

HTTP method policy:
- `GET`: read
- `POST`: create/commands
- `PUT`: full update
- `PATCH`: partial update
- `DELETE`: soft delete/archive

Example route registration pattern:

```go
func RegisterRoutes(r chi.Router, h *Handler) {
    r.Route("/api/v1/users", func(api chi.Router) {
        api.Post("/register", h.Register)
        api.Post("/login", h.Login)
        api.Get("/me", h.GetMe)
    })
}
```

## 4) Middleware Baseline

Apply these globally unless explicitly exempted:

- `RequestID`
- `Recoverer`
- `JWTAuth`
- `RBAC`
- `RateLimit`

Notes:
- Public endpoints (example: login/register, health) may bypass auth middleware.
- Protected endpoints must require `Authorization: Bearer <jwt>`.

## 5) Data and Persistence Rules

- Use GORM models with explicit table mapping when needed.
- Keep table/column names in `snake_case`.
- Use migrations for schema changes.
- Enforce constraints (`not null`, `unique`, FK, role checks).
- DELETE behavior must be soft delete using `gorm.DeletedAt`.

Model and DTO separation:
- Models represent persistence.
- DTOs represent API input/output.
- Do not expose internal/sensitive fields in response DTOs.

## 6) Error and Response Contract

- Return consistent JSON error shapes.
- Map domain errors to appropriate HTTP status codes.
- Validate input early at the handler boundary.
- Include request ID in logs and error traces when available.

## 7) Authentication and Authorization

- Validate JWT using Keycloak realm JWK.
- Use RBAC checks in middleware and service layer for sensitive flows.
- Admin-only routes must require role `admin`.
- Teacher/student capability checks must align with route intent.

## 8) Service Route Ownership

Each bounded context owns only its routes:

- User service: `/users`, `/students`, `/teachers`
- Learning content: `/courses`, `/modules`
- Progress: `/progress`
- Notification: `/notifications`
- Payment: `/plans`, `/subscriptions`, `/payments`, `/invoices`, `/coupons`
- Search: `/search`
- Admin: `/admin`

All prefixed by `/api/v1`.

## 9) Environment Variables

Use env-driven configuration. Minimum pattern:

```env
API_PREFIX=/api/v1
USER_SERVICE_PORT=8081
LEARNING_CONTENT_SERVICE_PORT=8082
PROGRESS_TRACKING_SERVICE_PORT=8083
NOTIFICATION_SERVICE_PORT=8084
PAYMENT_SERVICE_PORT=8085
SEARCH_SERVICE_PORT=8086
ADMIN_SERVICE_PORT=8087
KEYCLOAK_URL=http://localhost:8080
```

## 10) Testing Requirement

- Unit tests for handler, service, repository logic.
- Prefer table-driven tests.
- Keep test files as `*_test.go`.
- Add authorization tests for role-sensitive endpoints.
- Add integration tests for critical persistence workflows.

## 11) Definition of Done (Backend)

A backend feature is done only when:

- Route follows `/api/v1` and bounded context ownership
- Handler/service/repository boundaries are respected
- Input validation and error mapping are complete
- JWT/RBAC rules are enforced
- Migration and GORM model changes are aligned
- Tests are added/updated and passing

## 12) Business Logic Acceptance

Before finalizing implementation, verify:

- Role and ownership rules are explicitly enforced.
- Data changes are safe for retries and duplicate requests.
- Financial and progress-related mutations are auditable.
- Error responses support user-facing remediation paths.
- No endpoint breaks defined learner/teacher/admin journeys.
