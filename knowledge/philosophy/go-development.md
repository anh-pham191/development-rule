## Go Development Philosophy

This codebase follows these core principles to ensure clean, maintainable, and idiomatic Go:

### 1. Idiomatic

Write Go the way Go is meant to be written. Follow Effective Go and community conventions.

- ✓ DO: Use `errors.Is()` for sentinel errors, table-driven tests, small interfaces
- ✗ DON'T: Force patterns from other languages (inheritance, exceptions, getters/setters)

### 2. Concise

Keep code minimal without sacrificing clarity. Avoid unnecessary abstractions.

Functions should be short enough to hold in your head. If you can't see the entire function at once, or if you lose track of what variables hold, the function is probably doing too much. Deep nesting signals that the function is handling too many concerns. Consider extracting helper functions or restructuring.

- ✓ DO: Direct, clear function names that describe what they do; small functions with few variables
- ✗ DON'T: AbstractFactoryImpl patterns when simple functions suffice; functions that scroll for pages

### 3. Readable

Optimise for human understanding. Code should be self-explanatory.

Imagine a new developer joining the team tomorrow. They're competent but unfamiliar with this codebase. Could they read your function and understand its purpose within a few minutes? Write for the newcomer. Your future self, returning to this code in six months, will thank you.

- ✓ DO: Clear, descriptive names that convey intent; code that reads like prose
- ✗ DON'T: Cryptic abbreviations; code that requires tribal knowledge to understand

### 4. Explicit

Prefer clarity over implicit behavior. Make data flow obvious.

- ✓ DO: Return errors explicitly, show dependencies clearly
- ✗ DON'T: Hidden global state or magic side effects

### 5. Compositional

Use composition over inheritance. Build flexible systems with structs and interfaces.

- ✓ DO: Small interfaces, struct embedding, functional options
- ✗ DON'T: Deep inheritance hierarchies

### 6. Testable

Design for testing from the start. TDD drives better design.

- ✓ DO: Write test first, use interfaces for dependencies
- ✗ DON'T: Implement before test, create untestable global state

### 7. Pragmatic

Solve the problem at hand without unnecessary complexity. We value simplicity over cleverness, and prefer obvious over optimal.

- ✓ DO: Start simple, refactor when patterns emerge; write code that is immediately understandable
- ✗ DON'T: Over-engineer for hypothetical requirements; write "clever" code that requires explanation to understand

When optimisation requires sacrificing clarity, accompany it with a comment explaining why the trade-off was made.

### 8. Predictable

Code should behave as expected without surprises.

- ✓ DO: Pure functions where possible, immutable values
- ✗ DON'T: Functions with hidden side effects

### 9. Well-Commented

Well-written code is self-evident about *what* it does. Comments add value when they explain *why*: the context, constraints, or reasoning that isn't obvious from the code itself.

- ✓ DO: Explain business rules, non-obvious constraints, or the reasoning behind a design choice; note when code works around an external limitation
- ✗ DON'T: Narrate what the code does step-by-step; leave comments that duplicate the function name
