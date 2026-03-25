# Environment Guidelines

## Business Operation Profile

Environment configuration must protect business continuity and consistent user journeys.

Business-focused requirements:

- Service URLs and ports must remain stable per environment to avoid broken customer flows.
- API prefix `/api/v1` must remain consistent across services to reduce integration drift.
- Authentication configuration must fail closed (deny unauthorized access by default).
- Runtime settings must support incident response (observability, traceability, rollback).

Release safety rules:

- No production deployment without validated Keycloak and JWT configuration.
- No route prefix change without coordinated frontend and API client update.
- No service port/base URL drift without updating environment manifests and runbooks.

## Routing Configuration

### Guidelines
- Ensure all routes are defined in a centralized configuration file.
- Use environment variables to define base URLs for different environments (e.g., development, staging, production).
- Example environment variables:
  ```env
  REACT_APP_API_BASE_URL=https://api.example.com
  REACT_APP_ROUTING_MODE=hash
  ```
- Use a consistent naming convention for route paths.
- Document all routes and their purposes in the project documentation.

### Example Setup
```env
REACT_APP_API_BASE_URL=https://api.dev.example.com
REACT_APP_ROUTING_MODE=browser
```

## Routing for Services

### Guidelines
- Define one base URL and one port per service using environment variables.
- Keep a shared API prefix for all backend services: `/api/v1`.
- Use a predictable local port range for microservices to simplify local development.
- Ensure route ownership is clear: each bounded context exposes only its own routes.

### Service Routing and Port Matrix

| Service | Env Base URL Key | Default Local Port | Route Prefix / Main Routes |
| --- | --- | --- | --- |
| Frontend (Svelte) | FRONTEND_BASE_URL | 5173 | `/` |
| User Service | USER_SERVICE_BASE_URL | 8081 | `/api/v1/users`, `/api/v1/students`, `/api/v1/teachers` |
| Learning Content Service | LEARNING_CONTENT_SERVICE_BASE_URL | 8082 | `/api/v1/courses`, `/api/v1/modules` |
| Progress Tracking Service | PROGRESS_TRACKING_SERVICE_BASE_URL | 8083 | `/api/v1/progress` |
| Notification Service | NOTIFICATION_SERVICE_BASE_URL | 8084 | `/api/v1/notifications` |
| Payment Service | PAYMENT_SERVICE_BASE_URL | 8085 | `/api/v1/plans`, `/api/v1/subscriptions`, `/api/v1/payments`, `/api/v1/invoices`, `/api/v1/coupons` |
| Search Service | SEARCH_SERVICE_BASE_URL | 8086 | `/api/v1/search` |
| Admin Service | ADMIN_SERVICE_BASE_URL | 8087 | `/api/v1/admin` |
| Keycloak | KEYCLOAK_URL | 8080 | `/realms/{realm}`, `/protocol/openid-connect/*` |

### Example Setup
```env
API_PREFIX=/api/v1

FRONTEND_BASE_URL=http://localhost:5173

USER_SERVICE_PORT=8081
USER_SERVICE_BASE_URL=http://localhost:8081

LEARNING_CONTENT_SERVICE_PORT=8082
LEARNING_CONTENT_SERVICE_BASE_URL=http://localhost:8082

PROGRESS_TRACKING_SERVICE_PORT=8083
PROGRESS_TRACKING_SERVICE_BASE_URL=http://localhost:8083

NOTIFICATION_SERVICE_PORT=8084
NOTIFICATION_SERVICE_BASE_URL=http://localhost:8084

PAYMENT_SERVICE_PORT=8085
PAYMENT_SERVICE_BASE_URL=http://localhost:8085

SEARCH_SERVICE_PORT=8086
SEARCH_SERVICE_BASE_URL=http://localhost:8086

ADMIN_SERVICE_PORT=8087
ADMIN_SERVICE_BASE_URL=http://localhost:8087

KEYCLOAK_PORT=8080
KEYCLOAK_URL=http://localhost:8080
```

### Detailed Routes by Service

All backend routes are served under `${API_PREFIX}`.

#### User Service (port: `${USER_SERVICE_PORT}`)
- `${API_PREFIX}/users/register`
- `${API_PREFIX}/users/login`
- `${API_PREFIX}/users/refresh-token`
- `${API_PREFIX}/users/logout`
- `${API_PREFIX}/users/me`
- `${API_PREFIX}/users/{id}`
- `${API_PREFIX}/users/{id}/status`
- `${API_PREFIX}/users/{id}/settings`
- `${API_PREFIX}/students/{id}/enrollments`
- `${API_PREFIX}/students/{id}/enrollments/{courseId}`
- `${API_PREFIX}/teachers`
- `${API_PREFIX}/teachers/{id}`
- `${API_PREFIX}/teachers/{id}/classes`
- `${API_PREFIX}/teachers/{id}/classes/{classId}`
- `${API_PREFIX}/teachers/{id}/students/{studentId}/invite`
- `${API_PREFIX}/teachers/{id}/analytics`

#### Learning Content Service (port: `${LEARNING_CONTENT_SERVICE_PORT}`)
- `${API_PREFIX}/courses`
- `${API_PREFIX}/courses/{courseId}`
- `${API_PREFIX}/courses/{courseId}/publish`
- `${API_PREFIX}/courses/{courseId}/modules`
- `${API_PREFIX}/modules/{moduleId}`
- `${API_PREFIX}/modules/{moduleId}/lessons`

#### Progress Tracking Service (port: `${PROGRESS_TRACKING_SERVICE_PORT}`)
- `${API_PREFIX}/progress/events`
- `${API_PREFIX}/progress/users/{userId}`
- `${API_PREFIX}/progress/users/{userId}/courses/{courseId}`
- `${API_PREFIX}/progress/users/{userId}/streak`
- `${API_PREFIX}/progress/users/{userId}/badges`
- `${API_PREFIX}/progress/quiz-attempts`
- `${API_PREFIX}/progress/classes/{classId}/summary`
- `${API_PREFIX}/progress/teachers/{teacherId}/overview`

#### Notification Service (port: `${NOTIFICATION_SERVICE_PORT}`)
- `${API_PREFIX}/notifications`
- `${API_PREFIX}/notifications/users/{userId}`
- `${API_PREFIX}/notifications/{notificationId}/read`
- `${API_PREFIX}/notifications/bulk`
- `${API_PREFIX}/notifications/schedules`
- `${API_PREFIX}/notifications/schedules/{scheduleId}`

#### Payment Service (port: `${PAYMENT_SERVICE_PORT}`)
- `${API_PREFIX}/plans`
- `${API_PREFIX}/subscriptions`
- `${API_PREFIX}/subscriptions/{subscriptionId}`
- `${API_PREFIX}/payments/users/{userId}`
- `${API_PREFIX}/invoices/users/{userId}`
- `${API_PREFIX}/coupons/validate`
- `${API_PREFIX}/payments/webhooks`

#### Search Service (port: `${SEARCH_SERVICE_PORT}`)
- `${API_PREFIX}/search`
- `${API_PREFIX}/search/courses`
- `${API_PREFIX}/search/lessons`
- `${API_PREFIX}/search/teachers`
- `${API_PREFIX}/search/reindex`
- `${API_PREFIX}/search/reindex/courses/{courseId}`

#### Admin Service (port: `${ADMIN_SERVICE_PORT}`)
- `${API_PREFIX}/admin/users`
- `${API_PREFIX}/admin/users/{userId}/roles`
- `${API_PREFIX}/admin/users/{userId}/status`
- `${API_PREFIX}/admin/content/review`
- `${API_PREFIX}/admin/content/{contentId}/approve`
- `${API_PREFIX}/admin/content/{contentId}/reject`
- `${API_PREFIX}/admin/audit-logs`
- `${API_PREFIX}/admin/system/health`
- `${API_PREFIX}/admin/feature-flags/{flagKey}`

## Keycloak Configuration

### Guidelines
- Use environment variables to configure Keycloak settings.
- Example environment variables:
  ```env
  KEYCLOAK_URL=https://auth.example.com
  KEYCLOAK_REALM=myrealm
  KEYCLOAK_CLIENT_ID=myclientid
  ```
- Ensure sensitive information like client secrets is stored securely and not hardcoded.
- Use a library or SDK to integrate Keycloak with your application.

### Example Setup
```env
KEYCLOAK_URL=https://auth.dev.example.com
KEYCLOAK_REALM=devrealm
KEYCLOAK_CLIENT_ID=devclientid
```

## JWT Configuration

### Guidelines
- Use environment variables to configure JWT settings.
- Example environment variables:
  ```env
  JWT_SECRET=supersecretkey
  JWT_EXPIRATION=3600
  JWT_ISSUER=myapp
  ```
- Ensure the JWT secret is stored securely and not hardcoded.
- Use a library to handle JWT creation, validation, and decoding.
- Set appropriate expiration times for tokens.

### Example Setup
```env
JWT_SECRET=devsupersecretkey
JWT_EXPIRATION=3600
JWT_ISSUER=devapp
```

## Business Release Checklist

- All critical services are reachable with expected base URLs.
- Login and token validation flow works in target environment.
- Service-to-service and frontend-to-service routing resolve correctly.
- Monitoring captures failures on auth, payment, and progress endpoints.
- Environment changes are documented for support and operations teams.