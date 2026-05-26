---
name: rust-development
description: >
  Rust development philosophy, idioms, and patterns for writing clean, maintainable Rust code.
  Use when: (1) Working on Rust projects, (2) Writing new Rust code, (3) Reviewing Rust code,
  (4) Refactoring Rust code, (5) Configuring clippy or rustfmt, (6) Discussing Rust best
  practices. Covers ownership, error handling, testing, and tooling for stable Rust on the
  2021/2024 edition.
---

# Rust Development

Core principles for writing clean, maintainable, and idiomatic Rust code on current stable.

## Philosophy

### 1. Idiomatic
Write Rust the way Rust is meant to be written. Follow the Rust API Guidelines.
- DO: Use `Result` + `?`, iterators, `match` on enums, the newtype pattern
- DON'T: Force OO hierarchies, exceptions, or `unwrap()` everywhere

### 2. Concise
Keep code minimal without sacrificing clarity.
- DO: Iterator chains, `if let` / `let else`, small focused functions
- DON'T: Wrap trivial code in generic traits "just in case"

### 3. Readable
Optimise for human understanding. Code should be self-explanatory.
- DO: Descriptive names; meaningful lifetime and type parameter names
- DON'T: Cryptic abbreviations or lifetime soup

### 4. Explicit
Prefer clarity over implicit behaviour. Make ownership obvious.
- DO: Return `Result` explicitly; borrow with `&T` when ownership isn't needed
- DON'T: Hide work behind `Deref` magic or panic from library code

### 5. Compositional
Build through traits, generics, and small composable types.
- DO: Small focused traits, newtypes, builders for complex construction
- DON'T: Mega-traits with twenty methods

### 6. Testable
Design for testability from the start.
- DO: Write the test first; `#[cfg(test)] mod tests` next to code; inject behind traits
- DON'T: Hide logic in `main`; create untestable `static mut` globals

### 7. Safe / Ownership-Respecting
Lean on the borrow checker; `unsafe` is a last resort.
- DO: Model invariants in the type system; prefer borrowing over cloning
- DON'T: Sprinkle `unsafe` to silence the compiler; clone everywhere to dodge lifetimes

### 8. Zero-Cost
Trust Rust's zero-cost abstractions; profile before optimising.
- DO: Use iterators and generics freely; measure before tuning
- DON'T: Hand-unroll loops on a hunch

### 9. Fearless Concurrency
Use the type system to make concurrency safe.
- DO: Prefer message passing; pick one async runtime per binary
- DON'T: Mix async runtimes; block inside async tasks

### 10. Pragmatic
Solve the problem at hand without unnecessary complexity.
- DO: Start simple, refactor when patterns emerge
- DON'T: Over-engineer for hypothetical requirements

### 11. Predictable
Handle errors explicitly. No surprises.
- DO: Return `Result` with a specific error type; document panics
- DON'T: `unwrap()` in production paths; silently swallow errors

### 12. Tooling-Strict
Let `rustfmt` and `clippy` enforce the baseline.
- DO: Run `cargo fmt` and `cargo clippy -- -D warnings` in CI
- DON'T: Disable lints without a justification comment

## Error Handling

Use `Result<T, E>` with `?` to propagate. Prefer a typed error per crate.

For libraries, derive errors with `thiserror`:

```rust
use thiserror::Error;

#[derive(Debug, Error)]
pub enum ConfigError {
    #[error("config file not found at {path}")]
    NotFound { path: String },

    #[error("failed to parse config")]
    Parse(#[from] toml::de::Error),

    #[error("io error")]
    Io(#[from] std::io::Error),
}

pub fn load_config(path: &str) -> Result<Config, ConfigError> {
    let raw = std::fs::read_to_string(path)?;
    let config = toml::from_str(&raw)?;
    Ok(config)
}
```

For binaries / top-level glue, `anyhow` adds context cheaply:

```rust
use anyhow::{Context, Result};

fn run() -> Result<()> {
    let config = load_config("app.toml")
        .with_context(|| "loading application config")?;
    start_server(config).context("starting server")?;
    Ok(())
}
```

Rules of thumb:

- Library crates: typed errors via `thiserror`, never `anyhow` in public APIs.
- Binary crates: `anyhow` at the edges; typed errors inside modules.
- `unwrap()` / `expect()` are acceptable in tests and in `main` returning `Result`, rarely elsewhere.

## Testing

Unit tests live next to the code; integration tests live in `tests/`.

```rust
pub fn slugify(input: &str) -> String {
    input
        .to_lowercase()
        .chars()
        .map(|c| if c.is_alphanumeric() { c } else { '-' })
        .collect::<String>()
        .trim_matches('-')
        .to_string()
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn slugifies_input() {
        let cases = [
            ("Hello World", "hello-world"),
            ("Rust 2024!", "rust-2024"),
            ("  spaced  ", "spaced"),
        ];

        for (input, expected) in cases {
            assert_eq!(slugify(input), expected, "input = {input:?}");
        }
    }

    #[test]
    fn handles_empty_input() {
        assert_eq!(slugify(""), "");
    }
}
```

For parameterised tests with richer reporting, `rstest` is the community default:

```rust
use rstest::rstest;

#[rstest]
#[case("Hello World", "hello-world")]
#[case("Rust 2024!", "rust-2024")]
#[case("  spaced  ", "spaced")]
fn slugifies_input(#[case] input: &str, #[case] expected: &str) {
    assert_eq!(slugify(input), expected);
}
```

## Ownership & Borrowing

Pass borrows when you only need to read; pass owned values when you need to keep them.

```rust
// Borrow when reading
fn word_count(text: &str) -> usize {
    text.split_whitespace().count()
}

// Take ownership when storing
struct Document {
    body: String,
}

impl Document {
    fn new(body: String) -> Self {
        Self { body }
    }
}

// &mut when mutating in place
fn normalise(text: &mut String) {
    *text = text.trim().to_lowercase();
}
```

Prefer `&str` over `&String`, `&[T]` over `&Vec<T>` in function signatures:

```rust
// DO: accept the widest useful type
fn first_line(text: &str) -> Option<&str> {
    text.lines().next()
}

// DON'T: needlessly narrow the API
fn first_line_bad(text: &String) -> Option<&str> {
    text.lines().next()
}
```

The newtype pattern keeps domain values from being confused:

```rust
pub struct UserId(pub u64);
pub struct OrderId(pub u64);

fn fetch_user(id: UserId) -> Option<User> {
    // can't accidentally pass an OrderId
    // ...
    None
}
```

Reach for `Arc<Mutex<T>>` only when sharing mutable state across threads is genuinely needed; reach for `Rc<RefCell<T>>` even more rarely.

## Traits & Generics

Keep traits small and focused; accept generics, return concrete types where you can.

```rust
pub trait Repository<T> {
    type Error;
    fn find(&self, id: u64) -> Result<Option<T>, Self::Error>;
    fn save(&mut self, value: T) -> Result<(), Self::Error>;
}

// Accept any reader; return a concrete result
pub fn load<R: std::io::Read>(mut reader: R) -> std::io::Result<Vec<u8>> {
    let mut buf = Vec::new();
    reader.read_to_end(&mut buf)?;
    Ok(buf)
}
```

Use `dyn Trait` when the set of implementations is open or chosen at runtime; use generics when it's known at compile time and inlining matters.

## Async

Pick one runtime per binary (`tokio` is the common choice) and stick to it.

```rust
use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let (tx, mut rx) = tokio::sync::mpsc::channel(16);

    tokio::spawn(async move {
        for i in 0..5 {
            tx.send(i).await.ok();
            sleep(Duration::from_millis(10)).await;
        }
    });

    while let Some(value) = rx.recv().await {
        println!("got {value}");
    }

    Ok(())
}
```

Never call blocking code (`std::fs`, `std::thread::sleep`, CPU-heavy loops) directly inside an async task — use `tokio::fs`, `tokio::time::sleep`, or `tokio::task::spawn_blocking`.

## Tooling

The expected baseline on every Rust project:

```bash
# Format — must produce no diff
cargo fmt --all -- --check

# Lint — warnings are errors
cargo clippy --all-targets --all-features -- -D warnings

# Test — unit + integration + doctests
cargo test --all-features

# Doctests on public items
cargo test --doc
```

A reasonable `Cargo.toml` lint profile:

```toml
[lints.rust]
unsafe_code = "deny"
missing_docs = "warn"

[lints.clippy]
pedantic = { level = "warn", priority = -1 }
unwrap_used = "warn"
expect_used = "warn"
```

When suppressing a lint, explain why:

```rust
// reason: index is bounds-checked by the caller via `validate_range`
#[allow(clippy::indexing_slicing)]
fn fast_lookup(data: &[u8], i: usize) -> u8 {
    data[i]
}
```

## Unsafe

Every `unsafe` block must carry a `// SAFETY:` comment that justifies the invariants the caller is upholding.

```rust
/// Returns the byte at `index` without bounds checking.
///
/// # Safety
///
/// `index` must be less than `slice.len()`.
pub unsafe fn get_unchecked(slice: &[u8], index: usize) -> u8 {
    // SAFETY: caller guarantees `index < slice.len()`, so the pointer
    // arithmetic stays within the allocation.
    unsafe { *slice.as_ptr().add(index) }
}
```

If you can't write a convincing SAFETY comment, the code shouldn't be `unsafe`.
