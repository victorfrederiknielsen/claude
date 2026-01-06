---
name: go-executor
description: Implements Go features and services based on detailed plans. Use when you have a plan from go-planner and need to write the actual Go code. Follows idiomatic Go patterns and best practices.
---

# Go Implementation Executor

This skill implements Go features based on detailed implementation plans. It follows idiomatic Go patterns, emphasizes simplicity and clarity, and writes self-documenting code.

## Core Principles

### Idiomatic Go
- **Errors are values**: Handle explicitly, wrap with context
- **Accept interfaces, return structs**: For flexibility and clarity
- **Small, focused functions**: Single responsibility
- **Table-driven tests**: Test multiple cases efficiently
- **Goroutines and channels**: Use for concurrency, not parallelism by default

### Code Documentation Philosophy
- **NO comments for single-line changes**: Code should be self-documenting
- **DO document all exported APIs**: Package, types, functions, methods
- **DO document complex logic**: Non-obvious algorithms or business rules
- **DO use descriptive names**: Variables and functions explain themselves
- **DO write package documentation**: Explain package purpose in doc.go

### Code Style
- Follow gofmt, golint, go vet
- Use standard library when possible
- Prefer explicit over implicit
- Keep it simple and boring
- Early returns to reduce nesting

## Implementation Workflow

### 1. Review the Plan

Before starting implementation, carefully review:
- **Package structure**: What packages need to be created/modified?
- **Type definitions**: What types and interfaces are needed?
- **Error handling**: What can fail and how to handle it?
- **Concurrency**: Any goroutines or channels needed?
- **Testing strategy**: What tests are required?

### 2. Create Package Structure

Set up packages following the plan:

```go
// Package widget provides operations for managing user widgets.
//
// Widgets are configurable components that users can create, edit,
// and organize in their dashboards. This package handles widget
// CRUD operations, validation, and persistence.
package widget

import (
    "context"
    "errors"
    "fmt"
    "time"
)

var (
    ErrNotFound     = errors.New("widget not found")
    ErrInvalidInput = errors.New("invalid widget input")
    ErrUnauthorized = errors.New("unauthorized access")
)
```

### 3. Define Types and Interfaces

Create types following the plan:

```go
// Widget represents a user's configurable dashboard component.
type Widget struct {
    ID          string
    UserID      string
    Name        string
    Description string
    Settings    map[string]interface{}
    CreatedAt   time.Time
    UpdatedAt   time.Time
}

// Service handles widget business logic and orchestration.
type Service interface {
    Create(ctx context.Context, userID string, input CreateInput) (*Widget, error)
    Get(ctx context.Context, id string, userID string) (*Widget, error)
    Update(ctx context.Context, id string, userID string, input UpdateInput) (*Widget, error)
    Delete(ctx context.Context, id string, userID string) error
    List(ctx context.Context, userID string, opts ListOptions) ([]*Widget, error)
}

// Repository defines storage operations for widgets.
type Repository interface {
    Save(ctx context.Context, w *Widget) error
    FindByID(ctx context.Context, id string) (*Widget, error)
    FindByUserID(ctx context.Context, userID string, opts ListOptions) ([]*Widget, error)
    Delete(ctx context.Context, id string) error
}
```

**What NOT to do:**
```go
// Bad: Obvious comments on simple fields
type Widget struct {
    ID   string // The widget ID
    Name string // The widget name
}

// Good: Self-documenting structure
type Widget struct {
    ID   string
    Name string
}
```

### 4. Implement Service Logic

Implement business logic with proper error handling:

```go
type service struct {
    repo Repository
}

// NewService creates a new widget service with the given repository.
func NewService(repo Repository) Service {
    return &service{repo: repo}
}

// Create creates a new widget for the user.
// Returns ErrInvalidInput if the input fails validation.
func (s *service) Create(ctx context.Context, userID string, input CreateInput) (*Widget, error) {
    if err := input.Validate(); err != nil {
        return nil, fmt.Errorf("%w: %v", ErrInvalidInput, err)
    }

    widget := &Widget{
        ID:          generateID(),
        UserID:      userID,
        Name:        input.Name,
        Description: input.Description,
        Settings:    input.Settings,
        CreatedAt:   time.Now(),
        UpdatedAt:   time.Now(),
    }

    if err := s.repo.Save(ctx, widget); err != nil {
        return nil, fmt.Errorf("failed to save widget: %w", err)
    }

    return widget, nil
}

// Get retrieves a widget by ID.
// Returns ErrNotFound if the widget doesn't exist.
// Returns ErrUnauthorized if the widget doesn't belong to the user.
func (s *service) Get(ctx context.Context, id string, userID string) (*Widget, error) {
    widget, err := s.repo.FindByID(ctx, id)
    if err != nil {
        if errors.Is(err, ErrNotFound) {
            return nil, ErrNotFound
        }
        return nil, fmt.Errorf("failed to find widget: %w", err)
    }

    if widget.UserID != userID {
        return nil, ErrUnauthorized
    }

    return widget, nil
}
```

### 5. Implement Repository (Data Layer)

Implement storage operations:

```go
type postgresRepository struct {
    db *sql.DB
}

// NewPostgresRepository creates a new PostgreSQL-backed repository.
func NewPostgresRepository(db *sql.DB) Repository {
    return &postgresRepository{db: db}
}

// Save persists a widget to the database.
// If the widget already exists, it updates the existing record.
func (r *postgresRepository) Save(ctx context.Context, w *Widget) error {
    query := `
        INSERT INTO widgets (id, user_id, name, description, settings, created_at, updated_at)
        VALUES ($1, $2, $3, $4, $5, $6, $7)
        ON CONFLICT (id) DO UPDATE SET
            name = EXCLUDED.name,
            description = EXCLUDED.description,
            settings = EXCLUDED.settings,
            updated_at = EXCLUDED.updated_at
    `

    settingsJSON, err := json.Marshal(w.Settings)
    if err != nil {
        return fmt.Errorf("failed to marshal settings: %w", err)
    }

    _, err = r.db.ExecContext(ctx, query,
        w.ID, w.UserID, w.Name, w.Description,
        settingsJSON, w.CreatedAt, w.UpdatedAt,
    )
    if err != nil {
        return fmt.Errorf("failed to execute query: %w", err)
    }

    return nil
}

// FindByID retrieves a widget by its ID.
// Returns ErrNotFound if no widget exists with the given ID.
func (r *postgresRepository) FindByID(ctx context.Context, id string) (*Widget, error) {
    query := `
        SELECT id, user_id, name, description, settings, created_at, updated_at
        FROM widgets
        WHERE id = $1
    `

    var w Widget
    var settingsJSON []byte

    err := r.db.QueryRowContext(ctx, query, id).Scan(
        &w.ID, &w.UserID, &w.Name, &w.Description,
        &settingsJSON, &w.CreatedAt, &w.UpdatedAt,
    )
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, ErrNotFound
        }
        return nil, fmt.Errorf("failed to query widget: %w", err)
    }

    if err := json.Unmarshal(settingsJSON, &w.Settings); err != nil {
        return nil, fmt.Errorf("failed to unmarshal settings: %w", err)
    }

    return &w, nil
}
```

### 6. Handle Errors Idiomatically

Follow Go error handling patterns:

**Sentinel errors for expected cases:**
```go
var (
    ErrNotFound     = errors.New("widget not found")
    ErrInvalidInput = errors.New("invalid widget input")
)
```

**Wrap errors with context:**
```go
if err := s.repo.Save(ctx, widget); err != nil {
    return fmt.Errorf("failed to save widget %s: %w", widget.ID, err)
}
```

**Custom error types for rich errors:**
```go
// ValidationError represents a validation failure with field-level details.
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation failed for %s: %s", e.Field, e.Message)
}
```

**Check error types:**
```go
if errors.Is(err, ErrNotFound) {
    return nil, status.Error(codes.NotFound, "widget not found")
}

var validationErr *ValidationError
if errors.As(err, &validationErr) {
    return nil, status.Errorf(codes.InvalidArgument, validationErr.Message)
}
```

### 7. Implement Concurrency (When Needed)

Use goroutines and channels idiomatically:

```go
// ProcessBatch processes multiple widgets concurrently.
// Returns when all processing is complete or context is cancelled.
func (s *service) ProcessBatch(ctx context.Context, widgets []*Widget) error {
    errCh := make(chan error, len(widgets))

    for _, w := range widgets {
        w := w // capture loop variable

        go func() {
            errCh <- s.processWidget(ctx, w)
        }()
    }

    var errs []error
    for i := 0; i < len(widgets); i++ {
        if err := <-errCh; err != nil {
            errs = append(errs, err)
        }
    }

    if len(errs) > 0 {
        return fmt.Errorf("processing failed: %v", errs)
    }

    return nil
}
```

**With context and cleanup:**
```go
// WatchChanges watches for widget changes and sends updates to the channel.
// Stops watching when context is cancelled. Caller must consume the channel.
func (s *service) WatchChanges(ctx context.Context) (<-chan *Widget, error) {
    changes := make(chan *Widget)

    go func() {
        defer close(changes)

        ticker := time.NewTicker(5 * time.Second)
        defer ticker.Stop()

        for {
            select {
            case <-ctx.Done():
                return
            case <-ticker.C:
                widgets, err := s.getChangedWidgets(ctx)
                if err != nil {
                    return
                }

                for _, w := range widgets {
                    select {
                    case changes <- w:
                    case <-ctx.Done():
                        return
                    }
                }
            }
        }
    }()

    return changes, nil
}
```

### 8. Write Table-Driven Tests

Write comprehensive tests:

```go
func TestService_Create(t *testing.T) {
    tests := []struct {
        name      string
        userID    string
        input     CreateInput
        wantErr   error
        setupMock func(*mockRepository)
    }{
        {
            name:   "valid widget",
            userID: "user-123",
            input: CreateInput{
                Name:        "My Widget",
                Description: "A test widget",
            },
            wantErr: nil,
            setupMock: func(m *mockRepository) {
                m.On("Save", mock.Anything, mock.Anything).Return(nil)
            },
        },
        {
            name:   "empty name",
            userID: "user-123",
            input: CreateInput{
                Name:        "",
                Description: "A test widget",
            },
            wantErr: ErrInvalidInput,
            setupMock: func(m *mockRepository) {
                // No mock setup needed - validation fails before repo call
            },
        },
        {
            name:   "repository error",
            userID: "user-123",
            input: CreateInput{
                Name:        "My Widget",
                Description: "A test widget",
            },
            wantErr: ErrInvalidInput, // wrapped error
            setupMock: func(m *mockRepository) {
                m.On("Save", mock.Anything, mock.Anything).
                    Return(errors.New("database error"))
            },
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            repo := &mockRepository{}
            tt.setupMock(repo)

            svc := NewService(repo)

            widget, err := svc.Create(context.Background(), tt.userID, tt.input)

            if tt.wantErr != nil {
                require.Error(t, err)
                if !errors.Is(err, tt.wantErr) {
                    t.Errorf("expected error %v, got %v", tt.wantErr, err)
                }
                require.Nil(t, widget)
            } else {
                require.NoError(t, err)
                require.NotNil(t, widget)
                assert.Equal(t, tt.input.Name, widget.Name)
            }

            repo.AssertExpectations(t)
        })
    }
}
```

### 9. Write Benchmarks for Critical Code

Add benchmarks for performance-critical code:

```go
func BenchmarkService_List(b *testing.B) {
    repo := newMockRepositoryWithData(1000) // 1000 widgets
    svc := NewService(repo)
    ctx := context.Background()
    opts := ListOptions{Limit: 50}

    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        _, err := svc.List(ctx, "user-123", opts)
        if err != nil {
            b.Fatal(err)
        }
    }
}
```

## Documentation Guidelines

### DO Document These

**Package documentation (doc.go):**
```go
// Package widget provides operations for managing user widgets.
//
// Widgets are configurable dashboard components that users can create,
// customize, and organize. This package handles the full lifecycle of
// widgets including creation, retrieval, updates, and deletion.
//
// # Usage
//
// Create a service with a repository:
//
//     repo := widget.NewPostgresRepository(db)
//     svc := widget.NewService(repo)
//
// Create a widget:
//
//     input := widget.CreateInput{
//         Name:        "My Widget",
//         Description: "A custom widget",
//     }
//     w, err := svc.Create(ctx, userID, input)
//
// # Error Handling
//
// All service methods return wrapped errors. Use errors.Is to check
// for specific error types:
//
//     if errors.Is(err, widget.ErrNotFound) {
//         // handle not found
//     }
package widget
```

**Exported functions and methods:**
```go
// Create creates a new widget for the user.
//
// The widget name must be non-empty and no longer than 100 characters.
// Settings must be valid JSON when marshaled.
//
// Returns ErrInvalidInput if validation fails.
// Returns a wrapped error if the repository operation fails.
func (s *service) Create(ctx context.Context, userID string, input CreateInput) (*Widget, error) {
    // implementation
}
```

**Exported types:**
```go
// Widget represents a user's configurable dashboard component.
//
// Widgets contain user-defined settings stored as JSON and maintain
// creation and update timestamps for auditing.
type Widget struct {
    ID          string
    UserID      string
    Name        string
    Description string
    Settings    map[string]interface{}
    CreatedAt   time.Time
    UpdatedAt   time.Time
}
```

**Complex algorithms:**
```go
// calculateOptimalBatchSize determines the ideal batch size for processing
// based on available memory and item size. Uses a heuristic of targeting
// 80% memory utilization to leave headroom for GC.
func calculateOptimalBatchSize(availableMemory int64, itemSize int64) int {
    targetMemory := int64(float64(availableMemory) * 0.8)
    batchSize := int(targetMemory / itemSize)

    // Clamp between min and max
    const minBatch, maxBatch = 10, 1000
    if batchSize < minBatch {
        return minBatch
    }
    if batchSize > maxBatch {
        return maxBatch
    }

    return batchSize
}
```

### DO NOT Document These

**Self-explanatory operations:**
```go
// Bad: Obvious comment
widgets := make([]*Widget, 0, len(ids)) // Create slice to hold widgets

// Good: No comment
widgets := make([]*Widget, 0, len(ids))
```

**Loop variables:**
```go
// Bad: Unnecessary documentation
for i, widget := range widgets { // Iterate through widgets
    // ...
}

// Good: Self-documenting code
for i, widget := range widgets {
    // ...
}
```

**Simple assignments:**
```go
// Bad: Obvious comment
isActive := widget.Status == StatusActive // Check if active

// Good: Descriptive name, no comment
isActive := widget.Status == StatusActive
```

## Implementation Checklist

Before marking implementation complete:

- [ ] All packages from the plan are created/modified
- [ ] Types and interfaces are properly defined
- [ ] All exported APIs are documented
- [ ] Error handling follows Go idioms (wrap with context)
- [ ] No comments on single-line obvious code
- [ ] Table-driven tests are written
- [ ] Benchmarks added for performance-critical code
- [ ] Code follows gofmt, golint, go vet
- [ ] Context is used for cancellation where appropriate
- [ ] Resources are properly cleaned up
- [ ] No goroutine leaks
- [ ] Zero values are handled correctly

## Common Patterns

### HTTP Handler
```go
// HandleCreateWidget handles POST requests to create widgets.
func (h *Handler) HandleCreateWidget(w http.ResponseWriter, r *http.Request) {
    var input CreateInput
    if err := json.NewDecoder(r.Body).Decode(&input); err != nil {
        respondError(w, http.StatusBadRequest, "invalid request body")
        return
    }

    userID := getUserID(r.Context())

    widget, err := h.service.Create(r.Context(), userID, input)
    if err != nil {
        if errors.Is(err, ErrInvalidInput) {
            respondError(w, http.StatusBadRequest, err.Error())
            return
        }
        respondError(w, http.StatusInternalServerError, "internal error")
        return
    }

    respondJSON(w, http.StatusCreated, widget)
}
```

### Middleware
```go
// Authenticate validates the request has a valid auth token and adds
// the user ID to the request context.
func Authenticate(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        token := r.Header.Get("Authorization")
        if token == "" {
            http.Error(w, "missing auth token", http.StatusUnauthorized)
            return
        }

        userID, err := validateToken(token)
        if err != nil {
            http.Error(w, "invalid token", http.StatusUnauthorized)
            return
        }

        ctx := context.WithValue(r.Context(), userIDKey, userID)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}
```

### Worker Pool
```go
// ProcessWithWorkers processes items concurrently using a worker pool.
// Returns when all items are processed or context is cancelled.
func ProcessWithWorkers(ctx context.Context, items []Item, numWorkers int) error {
    jobs := make(chan Item, len(items))
    results := make(chan error, len(items))

    var wg sync.WaitGroup
    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for item := range jobs {
                results <- processItem(ctx, item)
            }
        }()
    }

    for _, item := range items {
        jobs <- item
    }
    close(jobs)

    wg.Wait()
    close(results)

    var errs []error
    for err := range results {
        if err != nil {
            errs = append(errs, err)
        }
    }

    if len(errs) > 0 {
        return fmt.Errorf("processing failed: %v", errs)
    }

    return nil
}
```

## Notes

- Follow idiomatic Go patterns consistently
- Write self-documenting code, avoid obvious comments
- Document all exported APIs with clear, helpful comments
- Handle errors explicitly, wrap with context
- Use table-driven tests for comprehensive coverage
- Keep functions small and focused
- Prefer composition over inheritance
- Use interfaces for abstraction and testing
- Clean up resources properly (defer, context cancellation)
