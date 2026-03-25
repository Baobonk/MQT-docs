# Backend Development Conventions

This document outlines the conventions and best practices for backend development in the English Learning Web Application. These conventions ensure consistency, maintainability, and adherence to the architecture.

## Business Logic Policy

All backend implementation decisions must preserve product behavior and business rules before technical optimization.

Required business invariants:

- Role invariants: only `student`, `teacher`, `admin` are valid role values.
- Ownership invariants: a teacher can modify only teacher-owned classes and content.
- Enrollment invariants: student enrollment changes must be validated against account status and plan constraints.
- Revenue invariants: payment webhooks must be idempotent and never create duplicate invoices/subscriptions.
- Audit invariants: admin and moderation actions must be traceable in logs.
- Recovery invariants: archive flows must use soft delete to support restore and investigations.

Decision rule:

- If a technical shortcut conflicts with domain correctness, domain correctness wins.

---

## General Guidelines

1. **Programming Language**: All backend services must be implemented in Go.
2. **Router (Hard Rule)**: Use **Chi Router** exclusively for HTTP routing.
    - Allowed: `github.com/go-chi/chi/v5`, `github.com/go-chi/chi/v5/middleware`
    - Not allowed: `http.ServeMux`, `gorilla/mux`, `gin`, `echo`, `fiber`, or mixed routing strategies
3. **Layered Architecture**:
   - **Handler Layer**: Handles HTTP requests and responses.
   - **Service Layer**: Contains business logic.
   - **Repository Layer**: Interacts with the database using GORM.
4. **Authentication**: Use Keycloak for authentication and JWT for stateless session management.
5. **Database**: Use PostgreSQL as the database for all services.
6. **API Versioning**: Each service must define its own versioning prefix (e.g., `/user/v1`, `/course/v1`).
7. **Error Handling**: Use standardized error responses with HTTP status codes.
8. **Middleware**: Apply common middleware for all services:
   - `RequestID`
   - `Recoverer`
   - `JWTAuth`
   - `RBAC`
   - `RateLimit`
9. **Role Enum Standard**: User role values are strictly `student`, `teacher`, and `admin`.
10. **Soft Delete Policy**: All DELETE operations must implement a soft delete mechanism using the `gorm.DeletedAt` field. Records must never be permanently removed from the database.

---

## Routing Conventions

1. **Base Path**: All routes must start with a service-specific versioned prefix (e.g., `/users/v1`, `/courses/v1`).
2. **HTTP Methods**:
   - Use appropriate HTTP methods for each operation:
     - `GET`: Retrieve resources.
     - `POST`: Create resources.
     - `PUT`: Update resources.
     - `PATCH`: Partially update resources.
     - `DELETE`: Remove resources.
3. **Path Parameters**:
   - Use `{id}` for resource identifiers.
   - Use descriptive names for nested resources (e.g., `{courseId}`, `{lessonId}`).
4. **Query Parameters**:
   - Use query parameters for filtering, sorting, and pagination.
   - Example: `GET /courses?level=A1&sort=asc&page=1`.
5. **Teacher and Student Routes**:
   - Use `/students` for student-specific routes.
   - Use `/teachers` for teacher-specific routes.
6. **Route Registration Pattern (Chi only)**:
   - Define service router as `func RegisterRoutes(r chi.Router, h *Handler)`.
    - Mount service router from composition root using `r.Mount("/users/v1", serviceRouter)` (replace `users` with the service name).
   - Group authorization using `r.Group(func(r chi.Router) { ... })` with role middleware.

### Minimum Chi Router Bootstrap

```go
package main

import (
    "net/http"

    "github.com/go-chi/chi/v5"
    chimw "github.com/go-chi/chi/v5/middleware"
)

func NewRouter() http.Handler {
    r := chi.NewRouter()
    r.Use(chimw.RequestID)
    r.Use(chimw.Recoverer)
    r.Use(chimw.RealIP)
    r.Use(chimw.Logger)

    r.Route("/users/v1", func(api chi.Router) {
        api.Get("/health", func(w http.ResponseWriter, r *http.Request) {
            w.WriteHeader(http.StatusOK)
            _, _ = w.Write([]byte("ok"))
        })
    })

    return r
}
```

---

## Code Structure

1. **Project Layout**:
   - Organize code by service.
   - Each service should have the following structure:
     ```
     /service-name
     ├── handlers/       # HTTP handlers
     ├── services/       # Business logic
     ├── repositories/   # Database access
     ├── models/         # Data models
     ├── middleware/     # Service-specific middleware
     ├── routes.go       # Route definitions
     └── main.go         # Service entry point
     ```

2. **File Naming**:
   - Use snake_case for file names.
   - Example: `user_service.go`, `progress_repository.go`.

3. **Function Naming**:
   - Use PascalCase for exported functions.
   - Use camelCase for unexported functions.

4. **Testing**:
   - Write unit tests for all layers.
   - Use table-driven tests for handlers and services.
   - Place tests in a `*_test.go` file.

---

## Middleware

1. **Global Middleware**:
   - Apply the following middleware globally:
     - `RequestID`: Adds a unique request ID to each request.
     - `Recoverer`: Recovers from panics and logs errors.
     - `JWTAuth`: Validates JWT tokens.
     - `RBAC`: Enforces role-based access control.
     - `RateLimit`: Limits the number of requests per user.

2. **Service-Specific Middleware**:
   - Add middleware specific to the service in the `middleware/` directory.

---

## Database Conventions

1. **ORM**: Use GORM for database interactions.
2. **Migrations**:
   - Use a migration tool (e.g., `golang-migrate`) for schema changes.
   - Store migration files in a `migrations/` directory.
3. **Naming**:
   - Use snake_case for table and column names.
   - Example: `users`, `lesson_content`.
4. **Relationships**:
   - Define relationships explicitly in GORM models.
   - Use foreign keys for referential integrity.

---

## Models and DTOs

### Models
- **Definition**: Models represent the database schema and are used for database interactions.
- **Conventions**:
  - Use GORM tags to define table and column mappings.
  - Define relationships explicitly (e.g., `hasMany`, `belongsTo`).
    - Use `snake_case` for table and column names.
    - Specify table source explicitly using `TableName() string` for every aggregate root model.
    - Prefer database constraints over implicit assumptions (`not null`, `uniqueIndex`, foreign keys).
    - For role values, use a check constraint to enforce `student`, `teacher`, or `admin`.

### Model-to-Table Mapping (Required)

| Model | Table | Primary Key | Notes |
| --- | --- | --- | --- |
| `User` | `users` | `id` | Contains role, profile, auth linkage |
| `TeacherProfile` | `teacher_profiles` | `id` | Teacher-specific metadata |
| `StudentEnrollment` | `student_enrollments` | `id` | Student-course relation |
| `Course` | `courses` | `id` | Owned by teacher |
| `Module` | `modules` | `id` | Belongs to course |
| `Lesson` | `lessons` | `id` | Belongs to module |
| `Quiz` | `quizzes` | `id` | Belongs to lesson |
| `ProgressEvent` | `progress_events` | `id` | User learning events |
| `Notification` | `notifications` | `id` | In-app/email events |
| `Subscription` | `subscriptions` | `id` | Billing lifecycle |
| `Invoice` | `invoices` | `id` | Billing documents |
- **Example**:
  ```go
  package models

    import (
            "time"

            "gorm.io/gorm"
    )

    const (
            UserRoleStudent = "student"
            UserRoleTeacher = "teacher"
            UserRoleAdmin   = "admin"
    )

  type User struct {
            ID        uint           `gorm:"primaryKey"`
            CreatedAt time.Time
            UpdatedAt time.Time
            DeletedAt gorm.DeletedAt `gorm:"index"`

            Username string `gorm:"size:50;not null;uniqueIndex"`
            Email    string `gorm:"size:255;not null;uniqueIndex"`
            Role     string `gorm:"size:20;not null;check:role IN ('student','teacher','admin')"`
            Password string `gorm:"size:255;not null"`
  }

  // TableName specifies the table name for the User model.
  func (User) TableName() string {
      return "users"
  }
  ```

### DTOs (Data Transfer Objects)
- **Definition**: DTOs are used to transfer data between layers, especially for API requests and responses.
- **Conventions**:
  - Use DTOs to decouple the API layer from the database schema.
  - Validate DTOs using libraries like `go-playground/validator`.
    - Keep DTOs transport-focused: no ORM tags, no DB behavior.
    - Split input and output DTOs (`CreateXRequest`, `XResponse`).
    - Never expose sensitive fields like `password_hash`, provider tokens, or internal flags.
- **Example**:
  ```go
  package dtos

  type RegisterUserDTO struct {
      Username string `json:"username" validate:"required,min=3,max=50"`
      Email    string `json:"email" validate:"required,email"`
    Role     string `json:"role" validate:"required,oneof=student teacher admin"`
      Password string `json:"password" validate:"required,min=8"`
  }

  type UserResponseDTO struct {
      ID       uint   `json:"id"`
      Username string `json:"username"`
      Email    string `json:"email"`
      Role     string `json:"role"`
  }
  ```

---

## Sample Code for Each Layer

## Business Readiness Checklist

Before merging backend work, confirm:

- Domain rules are enforced in service layer (not only handler input checks).
- Authorization and ownership checks are present for mutating operations.
- Route behavior reflects product intent and role capabilities.
- Error responses are actionable for frontend and support teams.
- Data writes are safe for retries and concurrent requests.
- Tests cover at least one critical business path and one failure path.

### Handler Layer
- **Responsibilities**: Handles HTTP requests and responses.
- **Example**:
  ```go
  package handlers

  import (
      "encoding/json"
      "net/http"

      "github.com/go-chi/chi/v5"
      "myapp/dtos"
      "myapp/services"
  )

  type UserHandler struct {
      UserService services.UserService
  }

  func (h *UserHandler) RegisterRoutes(r chi.Router) {
      r.Post("/users/register", h.RegisterUser)
  }

  func (h *UserHandler) RegisterUser(w http.ResponseWriter, r *http.Request) {
      var dto dtos.RegisterUserDTO
      if err := json.NewDecoder(r.Body).Decode(&dto); err != nil {
          http.Error(w, "invalid request", http.StatusBadRequest)
          return
      }

      user, err := h.UserService.RegisterUser(dto)
      if err != nil {
          http.Error(w, err.Error(), http.StatusInternalServerError)
          return
      }

      w.Header().Set("Content-Type", "application/json")
      w.WriteHeader(http.StatusCreated)
      json.NewEncoder(w).Encode(user)
  }
  ```

### Service Layer
- **Responsibilities**: Contains business logic.
- **Example**:
  ```go
  package services

  import (
      "errors"

      "myapp/dtos"
      "myapp/models"
      "myapp/repositories"
  )

  type UserService interface {
      RegisterUser(dto dtos.RegisterUserDTO) (*dtos.UserResponseDTO, error)
  }

  type userService struct {
      UserRepository repositories.UserRepository
      PasswordHasher PasswordHasher
  }

  type PasswordHasher interface {
      Hash(raw string) (string, error)
  }

  func (s *userService) RegisterUser(dto dtos.RegisterUserDTO) (*dtos.UserResponseDTO, error) {
      if dto.Role != models.UserRoleStudent && dto.Role != models.UserRoleTeacher && dto.Role != models.UserRoleAdmin {
          return nil, errors.New("invalid role")
      }

      hashed, err := s.PasswordHasher.Hash(dto.Password)
      if err != nil {
          return nil, errors.New("failed to hash password")
      }

      user := models.User{
          Username: dto.Username,
          Email:    dto.Email,
          Password: hashed,
          Role:     dto.Role,
      }

      if err := s.UserRepository.Create(&user); err != nil {
          return nil, errors.New("failed to create user")
      }

      return &dtos.UserResponseDTO{
          ID:       user.ID,
          Username: user.Username,
          Email:    user.Email,
          Role:     user.Role,
      }, nil
  }
  ```

### Repository Layer
- **Responsibilities**: Interacts with the database using GORM.
- **Example**:
  ```go
  package repositories

    import (
            "gorm.io/gorm"
            "myapp/models"
    )

  type UserRepository interface {
      Create(user *models.User) error
      FindByID(id uint) (*models.User, error)
  }

  type userRepository struct {
      DB *gorm.DB
  }

  func NewUserRepository(db *gorm.DB) UserRepository {
      return &userRepository{DB: db}
  }

  func (r *userRepository) Create(user *models.User) error {
      return r.DB.Create(user).Error
  }

  func (r *userRepository) FindByID(id uint) (*models.User, error) {
      var user models.User
      if err := r.DB.First(&user, id).Error; err != nil {
          return nil, err
      }
      return &user, nil
  }
  ```

### Business/Orchestration Layer (Optional for complex flows)
- **Responsibilities**: Coordinate cross-service workflows (e.g., create user + Keycloak provisioning + welcome notification) without adding transport concerns.
- **Example**:
    ```go
    package business

    import "myapp/dtos"

    type UserOnboardingService interface {
            OnboardTeacher(input dtos.RegisterUserDTO) (*dtos.UserResponseDTO, error)
    }
    ```

---

## API Documentation

1. **OpenAPI Specification**:
   - Document all APIs using OpenAPI (Swagger).
   - Store the specification in a `docs/` directory.
2. **Examples**:
   - Provide request and response examples for each endpoint.

---

## Logging and Monitoring

1. **Logging**:
   - Use a structured logging library (e.g., `logrus` or `zap`).
   - Include request IDs in all logs.
2. **Monitoring**:
   - Use Prometheus for metrics collection.
   - Expose a `/metrics` endpoint for each service.

---

## Deployment

1. **Containerization**:
   - Package each service as a Docker container.
   - Use multi-stage builds to optimize image size.
2. **Environment Variables**:
   - Use environment variables for configuration.
   - Example: `DB_HOST`, `KEYCLOAK_URL`.
3. **Orchestration**:
   - Use Kubernetes for service orchestration.
   - Define resources in YAML manifests.

---

By following these conventions, we ensure a consistent and maintainable backend architecture for the English Learning Web Application.