---
name: go-planner
description: Creates detailed Go implementation plans from Linear tickets or general Go requirements. Use when planning Go features, services, or backend work. Specializes in idiomatic Go patterns.
allowed-tools: Read, Grep, Glob, AskUserQuestion
---

# Go Implementation Planner

This skill creates comprehensive implementation plans for Go-based work. It analyzes requirements (from Linear tickets or direct requests) and generates detailed, actionable plans following idiomatic Go best practices.

## Idiomatic Go Principles

This planner follows Go best practices and emphasizes:

### Code Organization
- Package-oriented design
- Clear separation of concerns
- Interfaces for abstraction
- Table-driven tests

### Error Handling
- Errors are values
- Wrap errors with context
- Handle errors explicitly
- Fail fast when appropriate

### Concurrency
- Use goroutines and channels idiomatically
- Avoid shared memory, communicate by sharing
- Context for cancellation and timeouts
- sync primitives when needed

### Code Style
- Self-documenting code over comments for simple logic
- Document all exported APIs, functions, types
- Follow gofmt, golint, and go vet conventions
- Prefer simplicity and clarity

## Instructions

### 1. Analyze Requirements

From the input (Linear ticket output or direct request), identify:
- **Service responsibilities**: What does this service/package do?
- **API surface**: What functions/methods are exposed?
- **Data models**: What types and structures are needed?
- **Dependencies**: What external systems are involved?
- **Error scenarios**: What can go wrong and how to handle it?

### 2. Explore Existing Codebase

Use Read, Grep, and Glob to understand:
- **Package structure**: How is code organized?
- **Error handling patterns**: How are errors handled and wrapped?
- **Testing patterns**: What test helpers and fixtures exist?
- **Configuration approach**: Flags, env vars, config files?
- **Logging and observability**: What logging/metrics are used?

Search for:
```bash
# Find similar packages/services
find . -name "*.go" -path "*service*"

# Find error handling patterns
grep -r "fmt.Errorf\|errors.Wrap" .

# Find interface definitions
grep -r "type.*interface" .

# Find test patterns
grep -r "func Test" . --include="*_test.go"
```

### 3. Design Package Architecture

Plan the package structure:
- **Core domain logic**: Pure business logic
- **Interfaces**: Contracts for dependencies
- **Implementations**: Concrete types
- **Internal packages**: Private implementation details
- **Public API**: Exported functions and types

Consider:
- What should be exported vs internal?
- What interfaces are needed for testability?
- How to organize code for clarity?
- What packages already exist that can be reused?

### 4. Plan Data Models

For each type, determine:
- **Fields**: What data does it hold?
- **Methods**: What operations does it support?
- **Validation**: What constraints exist?
- **Serialization**: JSON, protobuf, etc.?
- **Zero values**: Are zero values valid?

Follow Go conventions:
- Simple structs over complex hierarchies
- Embed for composition
- Accept interfaces, return structs
- Pointer vs value receivers

### 5. Plan Error Handling

For each operation:
- **Error types**: Standard errors, custom types, or sentinel errors?
- **Error context**: What information helps debugging?
- **Error wrapping**: How to preserve error chains?
- **Recovery**: Can errors be recovered or should they fail fast?

Follow patterns:
```go
// Sentinel errors for expected cases
var ErrNotFound = errors.New("not found")

// Error wrapping for context
if err != nil {
    return fmt.Errorf("failed to process item %s: %w", id, err)
}

// Custom error types for rich errors
type ValidationError struct {
    Field string
    Err   error
}
```

### 6. Plan Concurrency

If concurrent operations are needed:
- **Goroutines**: Where should they be spawned?
- **Channels**: What data flows between goroutines?
- **Context**: How to handle cancellation and timeouts?
- **Synchronization**: Need for mutexes, wait groups, or sync.Once?
- **Resource cleanup**: How to ensure proper cleanup?

Avoid:
- Goroutine leaks
- Shared memory without synchronization
- Premature optimization with concurrency

### 7. Plan Testing Strategy

Include tests for:
- **Unit tests**: Individual functions and methods
- **Table-driven tests**: Multiple test cases efficiently
- **Integration tests**: Real dependencies when needed
- **Benchmarks**: Performance-critical code
- **Examples**: Documentation through runnable examples

### 8. Generate Implementation Plan

Create a structured plan with:

```markdown
# Go Implementation Plan: {Feature Name}

## Overview
- **Type**: {New Feature/Bug Fix/Refactor}
- **Scope**: {Package changes, new packages, etc.}
- **Complexity**: {Low/Medium/High}

## Requirements Analysis
{Summary of what needs to be built and why}

## Existing Codebase Patterns
- **Package structure**: {findings}
- **Error handling**: {patterns found}
- **Testing approach**: {tools and patterns}
- **Configuration**: {how config is managed}
- **Dependencies**: {existing dependencies and patterns}

## Package Architecture

### New Packages
1. `package_name` (path/to/package)
   - **Purpose**: {what it does}
   - **Exports**: {public API}
   - **Dependencies**: {what it depends on}
   - **Internal**: {private implementation details}

### Modified Packages
1. `existing_package` (path/to/package)
   - **Changes**: {what needs to change}
   - **Impact**: {what else is affected}

## Type Definitions

### New Types
```go
// Widget represents a user's widget with associated metadata.
type Widget struct {
    ID        string
    Name      string
    CreatedAt time.Time
}

// WidgetService handles widget operations.
type WidgetService interface {
    Create(ctx context.Context, w *Widget) error
    Get(ctx context.Context, id string) (*Widget, error)
    List(ctx context.Context, opts ListOptions) ([]*Widget, error)
}
```

### Modified Types
- `ExistingType`: {changes needed}

## Interface Design

{List interfaces for abstraction and testability}

```go
// Repository defines storage operations for widgets.
type Repository interface {
    Save(ctx context.Context, w *Widget) error
    Find(ctx context.Context, id string) (*Widget, error)
}
```

**Why**: {justification for each interface}

## Error Handling Plan

### Error Types
```go
var (
    ErrNotFound     = errors.New("widget not found")
    ErrInvalidInput = errors.New("invalid input")
)
```

### Error Wrapping Strategy
- Wrap errors with context using fmt.Errorf with %w
- Preserve error chains for errors.Is and errors.As
- Return sentinel errors for expected cases

## Implementation Steps

1. **Define core types and interfaces**
   - Create types in {package}/types.go
   - Define interfaces in {package}/service.go
   - Add validation methods

2. **Implement service logic**
   - Create implementation in {package}/impl.go
   - Handle business logic
   - Wrap errors with context

3. **Add data layer integration**
   - Implement repository interface
   - Handle database/storage operations
   - Add transaction support if needed

4. **Add HTTP/gRPC handlers** (if applicable)
   - Create handlers in {package}/handlers.go
   - Parse and validate requests
   - Map to/from domain types

5. **Wire up dependencies**
   - Update main.go or initialization code
   - Configure service dependencies
   - Set up middleware

6. **Write tests**
   - Unit tests for business logic
   - Table-driven tests for variations
   - Integration tests for full flows
   - Benchmarks for critical paths

## File Changes

### New Files
- `path/to/package/service.go` - Service interface and core logic
- `path/to/package/types.go` - Type definitions
- `path/to/package/repository.go` - Storage interface
- `path/to/package/impl.go` - Implementation
- `path/to/package/service_test.go` - Tests
- `path/to/package/doc.go` - Package documentation

### Modified Files
- `main.go` - Wire up new service
- `path/to/config/config.go` - Add configuration
- `path/to/api/routes.go` - Add endpoints (if applicable)

## Concurrency Plan

{If concurrent operations are needed}

### Goroutines
- **Where**: {spawn location}
- **Purpose**: {what concurrent work}
- **Cleanup**: {how to ensure cleanup}

### Channels
- **Type**: {channel type and buffer size}
- **Flow**: {producer → consumer}
- **Closure**: {who closes and when}

### Context Usage
- **Cancellation**: {how context propagates}
- **Timeouts**: {where timeouts are set}
- **Values**: {what context values if any}

## Testing Strategy

### Unit Tests
```go
func TestWidgetService_Create(t *testing.T) {
    tests := []struct {
        name    string
        input   *Widget
        want    error
    }{
        {"valid widget", &Widget{Name: "test"}, nil},
        {"empty name", &Widget{Name: ""}, ErrInvalidInput},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // test implementation
        })
    }
}
```

### Integration Tests
- {What to test with real dependencies}

### Benchmarks
- {Performance-critical code to benchmark}

## Technical Considerations

### Performance
- {considerations like pooling, caching, etc.}

### Observability
- {logging, metrics, tracing}

### Configuration
- {what needs to be configurable}

### Security
- {input validation, authentication, authorization}

## Potential Challenges
1. {Challenge}: {mitigation strategy}
2. {Challenge}: {mitigation strategy}

## Questions to Clarify
- {Any unclear requirements}
- {Design decisions needed}

## Next Steps
- Use the `go-executor` skill to implement this plan
- Review with team for feedback
```

## Planning Patterns by Task Type

### New Service/Package
1. Define service responsibilities and boundaries
2. Design public API (exported functions/types)
3. Plan dependencies and interfaces
4. Design error handling approach
5. Plan testing strategy
6. Consider observability needs

### Bug Fix
1. Reproduce and understand the bug
2. Identify root cause (logic, race, error handling?)
3. Plan minimal fix following existing patterns
4. Add test to prevent regression
5. Verify no breaking changes

### Refactoring
1. Understand current implementation
2. Identify issues (complexity, duplication, unclear responsibility)
3. Plan incremental refactoring steps
4. Ensure tests pass at each step
5. Update documentation

### Performance Optimization
1. Profile and identify bottlenecks
2. Plan optimizations (caching, concurrency, algorithms)
3. Avoid premature optimization
4. Add benchmarks to measure impact
5. Balance complexity vs gains

## Go Patterns to Favor

### ✅ Good Patterns
- Accept interfaces, return structs
- Errors are values, handle explicitly
- Table-driven tests
- Early returns to reduce nesting
- Context for cancellation and timeouts
- sync.Once for initialization
- Channels for coordination

### ❌ Patterns to Avoid
- Panic for expected errors (use errors)
- Ignoring errors
- Goroutine leaks
- Shared memory without synchronization
- Complex interface hierarchies
- Package cycles

## Documentation Guidelines

### What to Document
- **Package**: Package-level doc.go explaining purpose
- **Exported types**: What they represent
- **Exported functions**: What they do, params, returns, errors
- **Exported methods**: Same as functions
- **Examples**: How to use the API

### What NOT to Document
- Single-line obvious operations
- Private implementation details
- Loop variables and simple assignments
- Standard Go idioms

Example:
```go
// Good: Documents the API
// WidgetService provides operations for managing widgets.
type WidgetService interface {
    // Create creates a new widget with the given attributes.
    // Returns ErrInvalidInput if the widget is invalid.
    Create(ctx context.Context, w *Widget) error
}

// Bad: Documents obvious code
// Loop through widgets
for _, w := range widgets {
    // Set the status to active
    w.Status = "active"
}

// Good: Self-documenting code
for _, widget := range widgets {
    widget.Status = StatusActive
}
```

## Notes

- Always check existing patterns before introducing new ones
- Favor simple, boring solutions
- Consider error handling from the start
- Plan for testing, don't add it later
- This skill only plans - use `go-executor` to implement
