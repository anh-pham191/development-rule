## PHP Development Philosophy

This codebase follows these core principles to ensure clean, maintainable, and modern PHP.

**Assumptions:** This guide assumes modern PHP 8.x with Composer for dependency management and PSR-4 autoloading. Applications should commit `composer.lock` for reproducible builds; libraries should not.

### 1. Modern & Standards-Compliant

Embrace PHP 8.x features and follow PSR standards for interoperability and consistency.

- ✓ DO: Add `declare(strict_types=1);` to every file, use typed properties and union types, follow PSR-12 coding style
- ✗ DON'T: Use deprecated features, write untyped code, rely on implicit type coercion

### 2. Explicit

Make behavior clear and obvious. Avoid magic and implicit side effects.

- ✓ DO: Use type declarations, return types, and named arguments for clarity
- ✗ DON'T: Overuse magic methods (`__call`, `__get`), create hidden dependencies, rely on implicit behavior

### 3. Readable

Optimise for human understanding. Code should be self-documenting.

Imagine a new developer joining the team tomorrow. They're competent but unfamiliar with this codebase. Could they read your function and understand its purpose within a few minutes? Write for the newcomer. Your future self, returning to this code in six months, will thank you.

- ✓ DO: Use descriptive names, early returns, single responsibility methods; code that reads like prose
- ✗ DON'T: Write cryptic one-liners, use excessive abbreviations, nest deeply; code that requires tribal knowledge to understand

### 4. Well-Structured

Apply proper design principles. OOP is primary but functional patterns are valid where appropriate.

Functions and methods should be short enough to hold in your head. If you can't see the entire function at once, or if you lose track of what variables hold, the function is probably doing too much. Deep nesting signals that the function is handling too many concerns. Consider extracting helper methods or restructuring.

- ✓ DO: Use constructor property promotion, readonly properties, small focused classes; short methods with few variables
- ✗ DON'T: Create god classes, use global variables, rely on static methods for stateful logic; methods that scroll for pages

### 5. Compositional

Build flexible systems through composition and interfaces rather than deep class hierarchies.

- ✓ DO: Define interfaces for contracts, use dependency injection, prefer composition over inheritance
- ✗ DON'T: Create deep inheritance chains, use traits as a substitute for proper design

### 6. Testable

Design for testability from the start with proper dependency injection.

- ✓ DO: Write unit and integration tests, inject dependencies, use mocking/stubs
- ✗ DON'T: Create untestable static dependencies, rely on global state, hardcode external services

### 7. Secure

Follow security best practices built into modern PHP and frameworks.

- ✓ DO: Use prepared statements or ORM, validate input, escape output contextually
- ✗ DON'T: Concatenate SQL queries, trust user input, expose sensitive data in responses

### 8. Pragmatic

Solve the problem at hand without unnecessary complexity. We value simplicity over cleverness, and prefer obvious over optimal.

- ✓ DO: Start simple, refactor when patterns emerge, profile before optimising; write code that is immediately understandable
- ✗ DON'T: Over-engineer for hypothetical requirements, prematurely optimise; write "clever" code that requires explanation to understand

When optimisation requires sacrificing clarity, accompany it with a comment explaining why the trade-off was made.

### 9. Predictable

Code should behave as expected without surprises. Handle errors explicitly.

- ✓ DO: Throw specific exceptions, use custom exception hierarchies, document thrown exceptions with `@throws`
- ✗ DON'T: Silently swallow errors, use exceptions for flow control, leave error states ambiguous

### 10. Statically Analysed

Use static analysis and linting to catch errors before runtime.

- ✓ DO: Run PHPStan at level 6+, use phpcs with PSR-12, leverage generics and array shapes in docblocks for PHPStan
- ✗ DON'T: Ignore static analysis warnings, disable rules without justification, skip CI checks

### 11. Well-Commented

Well-written code is self-evident about *what* it does. Comments add value when they explain *why*: the context, constraints, or reasoning that isn't obvious from the code itself.

- ✓ DO: Explain business rules, non-obvious constraints, or the reasoning behind a design choice; note when code works around an external limitation
- ✗ DON'T: Narrate what the code does step-by-step; leave comments that duplicate the function name

---

### Framework Usage

When using frameworks (Laravel, Symfony, etc.), follow their conventions while maintaining clean architecture boundaries.

- ✓ DO: Follow framework conventions, use built-in features (DI, ORM, validation), extend at designated points
- ✗ DON'T: Reinvent framework features, tightly couple business logic to framework internals, bypass framework patterns
