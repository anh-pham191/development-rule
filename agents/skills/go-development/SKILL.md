---
name: go-development
description: >
  Go development philosophy, idioms, and patterns for writing clean, maintainable Go code.
  Use when: (1) Working on Go projects, (2) Writing new Go code, (3) Reviewing Go code,
  (4) Refactoring Go code, (5) Discussing Go best practices. Provides idiomatic Go patterns
  and principles that apply to any Go codebase.
---

# Go Development

Core principles for writing clean, maintainable, and idiomatic Go code.

## Philosophy

### 1. Idiomatic
Write Go the way Go is meant to be written. Follow Effective Go and community conventions.
- DO: Use `errors.Is()` for sentinel errors, table-driven tests, small interfaces
- DON'T: Force patterns from other languages (inheritance, exceptions, getters/setters)

### 2. Concise
Keep code minimal without sacrificing clarity. Avoid unnecessary abstractions.
- DO: Direct, clear function names that describe what they do
- DON'T: AbstractFactoryImpl patterns when simple functions suffice

### 3. Readable
Optimize for human understanding. Code should be self-explanatory.
- DO: Clear, descriptive names that convey intent
- DON'T: Cryptic abbreviations or clever one-liners

### 4. Explicit
Prefer clarity over implicit behavior. Make data flow obvious.
- DO: Return errors explicitly, show dependencies clearly
- DON'T: Hidden global state or magic side effects

### 5. Compositional
Use composition over inheritance. Build flexible systems with structs and interfaces.
- DO: Small interfaces, struct embedding, functional options
- DON'T: Deep inheritance hierarchies

### 6. Testable
Design for testing from the start. TDD drives better design.
- DO: Write test first, use interfaces for dependencies
- DON'T: Implement before test, create untestable global state

### 7. Pragmatic
Solve the problem at hand without unnecessary complexity.
- DO: Start simple, refactor when patterns emerge
- DON'T: Over-engineer for hypothetical requirements

### 8. Predictable
Code should behave as expected without surprises.
- DO: Pure functions where possible, immutable values
- DON'T: Functions with hidden side effects

## Error Handling

For detailed error patterns, see [error-handling.md](references/error-handling.md).

Quick reference:
```go
// Wrap errors with context using %w
if err != nil {
    return fmt.Errorf("loading config: %w", err)
}

// Check error types with errors.Is/As
if errors.Is(err, ErrNotFound) {
    // handle not found
}
```

## Testing

For detailed testing patterns, see [testing.md](references/testing.md).

Quick reference:
```go
// Table-driven tests
tests := []struct {
    name    string
    input   string
    want    string
    wantErr bool
}{
    {"valid input", "hello", "HELLO", false},
    {"empty input", "", "", true},
}

for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
        got, err := Transform(tt.input)
        if (err != nil) != tt.wantErr {
            t.Errorf("error = %v, wantErr %v", err, tt.wantErr)
        }
        if got != tt.want {
            t.Errorf("got %q, want %q", got, tt.want)
        }
    })
}
```

## Interface Design

Keep interfaces small and focused:

```go
// Good: small, focused interface
type Reader interface {
    Read(p []byte) (n int, err error)
}

// Accept interfaces, return structs
func Process(r Reader) (*Result, error) {
    // ...
}
```
