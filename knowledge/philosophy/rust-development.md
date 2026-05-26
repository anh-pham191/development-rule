## Rust Development Philosophy

This codebase follows these core principles to ensure clean, maintainable, and idiomatic Rust.

**Assumptions:** This guide assumes current stable Rust (managed via `rustup`) with Cargo as the build tool, and the 2021 or 2024 edition. Binary crates should commit `Cargo.lock` for reproducible builds; library crates should not. `rustfmt` and `clippy` are expected to run cleanly in CI.

### 1. Idiomatic

Write Rust the way Rust is meant to be written. Follow the Rust API Guidelines and community conventions.

- ✓ DO: Use `Result<T, E>` and `?` for fallible operations, iterators over manual loops, `match` on enums, newtypes for domain values
- ✗ DON'T: Force patterns from other languages (deep OO hierarchies, exceptions, nullable everywhere, `unwrap()` in production paths)

### 2. Concise

Keep code minimal without sacrificing clarity. Avoid unnecessary abstractions.

Functions should be short enough to hold in your head. If you can't see the entire function at once, or if you lose track of what variables hold, the function is probably doing too much. Deep nesting signals that the function is handling too many concerns. Consider extracting helper functions or restructuring.

- ✓ DO: Use iterator chains, `if let` / `let else` for early exits, small focused functions with descriptive names
- ✗ DON'T: Wrap trivial code in traits and generics "just in case"; write functions that scroll for pages

### 3. Readable

Optimise for human understanding. Code should be self-explanatory.

Imagine a new developer joining the team tomorrow. They're competent but unfamiliar with this codebase. Could they read your function and understand its purpose within a few minutes? Write for the newcomer. Your future self, returning to this code in six months, will thank you.

- ✓ DO: Clear, descriptive names that convey intent; code that reads like prose; meaningful lifetime and type parameter names where `'a` or `T` would obscure intent
- ✗ DON'T: Cryptic abbreviations; lifetime soup; code that requires tribal knowledge to understand

### 4. Explicit

Prefer clarity over implicit behaviour. Make data flow and ownership obvious.

- ✓ DO: Return `Result` explicitly, prefer `&T` / `&mut T` over `Clone` when borrowing suffices, name the error type
- ✗ DON'T: Hide work behind `Deref` coercion magic, use `Default` to silently paper over missing fields, panic from library code

### 5. Compositional

Build flexible systems with traits, generics, and small composable types. Prefer composition over inheritance (Rust has none anyway).

- ✓ DO: Small focused traits, trait objects (`dyn Trait`) where dynamism is needed, the newtype pattern, builder pattern for complex construction
- ✗ DON'T: Mega-traits with twenty methods; reach for generics when a function pointer or closure would do

### 6. Testable

Design for testing from the start. TDD drives better design.

- ✓ DO: Write the test first, keep unit tests next to the code in `#[cfg(test)] mod tests`, put integration tests under `tests/`, inject dependencies behind traits
- ✗ DON'T: Implement before test, hide logic in `main`, create untestable global state with `static mut` or `lazy_static`

### 7. Safe / Ownership-Respecting

Lean on the borrow checker rather than fighting it. Unsafe code is a last resort, not a shortcut.

- ✓ DO: Model invariants in the type system, prefer ownership and borrowing over reference counting, use `Arc<Mutex<T>>` only where shared mutable state is genuinely required
- ✗ DON'T: Sprinkle `unsafe` to silence the compiler; reach for `Rc<RefCell<T>>` to avoid thinking about ownership; clone everywhere to dodge lifetimes

### 8. Zero-Cost

Trust Rust's zero-cost abstractions. Write expressive code first; only sacrifice clarity for measured performance wins.

- ✓ DO: Use iterators, generics, and traits freely — the compiler will inline; profile before micro-optimising
- ✗ DON'T: Hand-unroll loops or hoist allocations on a hunch; assume a `Vec` is too slow without measuring

### 9. Fearless Concurrency

Use Rust's type system to make concurrency safe. `Send` and `Sync` are tools, not obstacles.

- ✓ DO: Prefer message passing (`std::sync::mpsc`, `tokio::sync`) over shared state; use `Arc` for shared ownership across threads; pick one async runtime per binary
- ✗ DON'T: Mix async runtimes; block inside async tasks; reach for `unsafe impl Send` without proving the invariant

### 10. Pragmatic

Solve the problem at hand without unnecessary complexity. We value simplicity over cleverness, and prefer obvious over optimal.

- ✓ DO: Start simple, refactor when patterns emerge; reach for a third-party crate only when it earns its place
- ✗ DON'T: Over-engineer for hypothetical requirements; write "clever" type-level code that requires a PhD to read

When optimisation requires sacrificing clarity, accompany it with a comment explaining why the trade-off was made.

### 11. Predictable

Code should behave as expected without surprises. Handle errors explicitly.

- ✓ DO: Return `Result` with a specific error type, use `thiserror` for library errors and `anyhow` for binary glue, document panics with `# Panics`
- ✗ DON'T: `unwrap()` / `expect()` in non-test code without justification; silently swallow errors with `let _ = ...`; rely on `panic!` for control flow

### 12. Tooling-Strict

Let the tools enforce the baseline so reviews can focus on design.

- ✓ DO: Run `cargo fmt` and `cargo clippy -- -D warnings` in CI, enable relevant lint groups (`clippy::pedantic` where sensible), keep `cargo test` green on every commit
- ✗ DON'T: Disable lints without a `// reason = "..."` comment; commit code that `rustfmt` would change; ignore deprecation warnings

### 13. Well-Commented

Well-written code is self-evident about *what* it does. Comments add value when they explain *why*: the context, constraints, or reasoning that isn't obvious from the code itself.

- ✓ DO: Write `///` doc comments on public items with examples that `cargo test` runs; explain business rules, non-obvious invariants, and the reasoning behind unsafe blocks (`// SAFETY: ...`)
- ✗ DON'T: Narrate what the code does step-by-step; leave comments that duplicate the function name; ship `unsafe` without a SAFETY justification
