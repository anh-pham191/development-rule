## C Development Philosophy

This codebase follows these core principles to ensure safe, portable, and maintainable C.

**Assumptions:** This guide assumes modern C (C11 / C17 / C23 where available), CMake or Make for builds, AddressSanitizer/UBSan/MSan in CI, static analysers (clang-tidy, scan-build) on every commit, and that the codebase compiles cleanly with `-Wall -Wextra -Werror` (or the platform equivalent). C is a small language with sharp edges; the rules below exist to keep those edges away from your fingers.

### 1. Readable

Optimise for human understanding. Code should be self-explanatory.

Imagine a new developer joining the team tomorrow. They're competent but unfamiliar with this codebase. Could they read your function and understand its purpose within a few minutes? Write for the newcomer. Your future self, returning to this code in six months, will thank you.

- ✓ DO: Name functions and variables for what they mean (`parse_http_header`, `bytes_read`); keep one logical concept per translation unit
- ✗ DON'T: Lean on single-letter names, clever macros, or three-deep pointer arithmetic in place of a helper function

### 2. Explicit

Say what you mean. C will not infer intent on your behalf, and silent assumptions become bugs.

- ✓ DO: Use fixed-width types (`uint32_t`, `int64_t`); cast deliberately; check every return value; document ownership in the header
- ✗ DON'T: Rely on implicit int promotion, default argument promotions, or "it happens to work on x86_64"

### 3. Concise

Keep functions small and focused.

Functions should be short enough to hold in your head. If you can't see the entire function at once, or if you lose track of what variables hold, the function is probably doing too much. Deep nesting signals that the function is handling too many concerns. Consider extracting helper functions or restructuring.

- ✓ DO: Extract helpers, keep cyclomatic complexity low, use early returns at the top of a function for input validation
- ✗ DON'T: Write 400-line functions with nested switch statements and seven levels of indentation

### 4. Compositional

Build programs from small, independent modules with narrow interfaces.

- ✓ DO: Use opaque pointers (`typedef struct foo foo_t;` in the header, definition in the `.c` file); keep translation units small; expose only what callers need
- ✗ DON'T: Spread one logical module across the global namespace, or expose struct internals just because it's convenient

### 5. Memory-Safe

Own every allocation. Free every allocation. Make ownership unambiguous at the API boundary.

- ✓ DO: Pair every `malloc`/`calloc`/`strdup` with a matching `free`; document ownership in the function comment (`/* caller owns the returned buffer */`); prefer arena or pool allocators for short-lived data
- ✗ DON'T: Return a heap pointer without telling the caller they own it, free memory the API doesn't own, or use a pointer after `free`

### 6. Undefined-Behaviour Aware

The C standard leaves a great deal undefined. Treat undefined behaviour as a bug, not a feature.

- ✓ DO: Check for signed overflow before it happens; respect strict aliasing (`memcpy` between dissimilar types); always initialise locals before reading them
- ✗ DON'T: Cast a `float*` to `int*` and dereference, shift by a width >= the type size, or assume `INT_MAX + 1` wraps

### 7. Portable

Don't bake host assumptions into the code.

- ✓ DO: Use `<stdint.h>` fixed-width types; use `size_t` for sizes and `ptrdiff_t` for pointer differences; check endianness explicitly when serialising; respect alignment
- ✗ DON'T: Assume `int` is 32 bits, `long` is 64 bits, structs have no padding, or that the host is little-endian

### 8. Defensive at Boundaries

Validate at the edge of your module; trust the inside.

- ✓ DO: Null-check pointer parameters in public functions; check sizes before copying; return clear error codes for invalid input
- ✗ DON'T: Sprinkle defensive checks through every internal helper — that just hides the real contract and inflates the binary

### 9. Testable

If you can't test it, you can't trust it.

- ✓ DO: Write unit tests with Criterion, Unity, or plain `assert`; run them under AddressSanitizer and UBSan; cover the error paths, not just the happy path
- ✗ DON'T: Write a 2,000-line module with no entry point smaller than `main`, or skip testing the `goto cleanup` branches

### 10. Tooled

The compiler and the sanitisers are your best reviewers. Use them.

- ✓ DO: Build with `-Wall -Wextra -Werror -Wshadow -Wconversion`; run AddressSanitizer + UndefinedBehaviorSanitizer in CI; run `clang-tidy` and `scan-build` on every commit
- ✗ DON'T: Disable warnings to silence them, ship a binary that's never been sanitiser-tested, or skip static analysis because "it's noisy"

### 11. Pragmatic

Solve the problem at hand without unnecessary complexity. We value simplicity over cleverness, and prefer obvious over optimal.

- ✓ DO: Profile before optimising; if you must use a SIMD intrinsic or a hand-rolled hot loop, hide it behind a clear function name and a comment explaining why
- ✗ DON'T: Micro-optimise readable code on a hunch, or scatter `restrict`, `__builtin_expect`, and inline assembly through code that isn't on the hot path

When optimisation requires sacrificing clarity, accompany it with a comment explaining why the trade-off was made.

### 12. Predictable

Same input, same output. The same call should not behave differently on the second run.

- ✓ DO: Initialise every local before reading it; make cleanup paths idempotent (`free(NULL)` is safe by design); avoid global mutable state; pass clocks and randomness as parameters when testability matters
- ✗ DON'T: Rely on the contents of uninitialised memory; let a function's behaviour depend on a hidden `static` cache; depend on undefined evaluation order in expressions like `f(i++, i++)`

### 13. Well-Commented

Well-written code is self-evident about *what* it does. Comments add value when they explain *why*: the context, constraints, or reasoning that isn't obvious from the code itself.

- ✓ DO: Document ownership, lifetime, threading assumptions, and any non-obvious invariant at the top of each public function
- ✗ DON'T: Write `/* increment i */` above `i++`, or let a Doxygen block grow stale next to a function whose signature has changed
