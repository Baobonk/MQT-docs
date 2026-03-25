# Svelte Frontend Instruction

This document defines implementation rules for the Svelte 4 frontend in this project.

## Business Outcome Requirement

Every frontend change must make business workflows clearer, safer, and easier to complete:

- student journey: discover, enroll, learn, track progress
- teacher journey: create content, manage classes, monitor outcomes
- admin journey: govern users, moderation, and feature controls
- payer journey: select plan, subscribe, and review billing records

## 1) Core Stack and UI Contract

- Framework: Svelte 4
- Frontend communicates with backend microservices via REST under `/api/v1`
- Authentication is JWT bearer token based, integrated with Keycloak-backed backend auth

Primary goals:
- Keep UI reactive and accessible
- Keep API integration typed and centralized
- Preserve clear boundaries between pages, components, and API client code

## 2) Project Structure Guideline

Recommended layout:

```text
src/
  app.html
  main.js
  routes/
  lib/
    components/
    stores/
    api/
    utils/
  styles/
```

Conventions:
- Route-level features in `routes/`
- Reusable UI in `lib/components/`
- Shared state in `lib/stores/`
- HTTP clients and endpoint wrappers in `lib/api/`

## 3) API Integration Rules

- Use one centralized API base URL from environment variables.
- Prefix all backend requests with `/api/v1`.
- Attach `Authorization: Bearer <jwt>` header for protected endpoints.
- Keep endpoint wrappers per bounded context (users, courses, progress, etc.).

Example environment setup:

```env
VITE_API_PREFIX=/api/v1
VITE_USER_SERVICE_BASE_URL=http://localhost:8081
VITE_LEARNING_CONTENT_SERVICE_BASE_URL=http://localhost:8082
VITE_PROGRESS_TRACKING_SERVICE_BASE_URL=http://localhost:8083
VITE_NOTIFICATION_SERVICE_BASE_URL=http://localhost:8084
VITE_PAYMENT_SERVICE_BASE_URL=http://localhost:8085
VITE_SEARCH_SERVICE_BASE_URL=http://localhost:8086
VITE_ADMIN_SERVICE_BASE_URL=http://localhost:8087
```

## 4) Auth and Session Handling

- Login flow calls backend auth endpoints (which integrate with Keycloak).
- Store access token securely in frontend state strategy agreed by the team.
- Add token automatically through a single request helper/interceptor.
- On `401`, trigger refresh/re-auth strategy and redirect when needed.
- Never hardcode secrets in frontend code.

## 5) Route and Feature Mapping

Frontend screens should align with bounded contexts:

- User area: profile, settings, enrollments
- Teacher area: classes, students, analytics
- Learning content: courses/modules/lessons
- Progress: dashboard, streak, badges
- Notifications: inbox and schedule views
- Payments: plans, subscriptions, invoices
- Search: global search and filtered results
- Admin: moderation, users, feature flags, audit views

## 6) State Management

- Use Svelte stores for shared client state.
- Keep server data state separate from UI-only state.
- Avoid duplicating fetched data across multiple stores.
- Normalize data where repeated reads are expected.

Suggested stores:
- `authStore`: user identity, token, roles
- `appConfigStore`: API URLs, runtime flags
- `uiStore`: global loading/error/toast state

## 7) Error Handling and UX

- Show friendly messages for expected failures.
- Surface validation errors near form fields.
- Handle network and timeout failures gracefully.
- Provide empty/loading/error states for all async views.
- Respect backend status codes and error semantics.

## 8) Role-Aware UI

Role model is strict: `student`, `teacher`, `admin`.

Rules:
- Hide or disable unauthorized actions in UI.
- Guard routes by role before rendering privileged screens.
- Still rely on backend RBAC as final authority.

## 9) Performance and Maintainability

- Prefer code-splitting for route-level modules.
- Keep components small and composable.
- Avoid heavy business logic in component markup.
- Move API/business transformations to helper modules.

## 10) Testing Requirement

- Unit test critical stores and utility functions.
- Component tests for key user flows (auth, enrollments, payments).
- Integration tests for role-based route protection.
- Add regression tests for known UI/API edge cases.

## 11) Definition of Done (Frontend)

A frontend feature is done only when:

- It uses centralized API clients and env config
- JWT header handling and auth edge cases are covered
- Role-based visibility and route guards are implemented
- Loading/empty/error states are present
- Tests are added/updated for critical paths
- UX and accessibility checks are completed

## 12) Business Logic Acceptance

Before marking a frontend task complete, confirm:

- Role-based screens and actions match business permissions.
- User can recover from expected failures (auth expiry, validation, network errors).
- Key conversion actions are unambiguous (enroll, subscribe, publish, approve).
- UI labels and states reflect domain language and product intent.
- No change blocks critical revenue or retention journeys.
