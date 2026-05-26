## Python Development Philosophy

This codebase follows these core principles to ensure clean, maintainable, and idiomatic Python.

**Assumptions:** This guide assumes modern Python 3.12+ with a managed virtual environment and a lockfile-based dependency manager (`uv` preferred, `poetry` acceptable). Applications should commit the lockfile (`uv.lock` or `poetry.lock`) for reproducible builds; libraries should pin only minimum compatible versions in `pyproject.toml`.

### 1. Pythonic

Write Python the way Python is meant to be written. Follow PEP 8, PEP 20 (The Zen of Python), and community conventions.

- ✓ DO: Use list/dict/set comprehensions, context managers, `enumerate`, `zip`, dataclasses, `pathlib`, f-strings
- ✗ DON'T: Force patterns from other languages (Java-style getters/setters, manual index loops, `os.path` string juggling)

### 2. Concise

Keep code minimal without sacrificing clarity. Avoid unnecessary abstractions.

Functions should be short enough to hold in your head. If you can't see the entire function at once, or if you lose track of what variables hold, the function is probably doing too much. Deep nesting signals that the function is handling too many concerns. Consider extracting helper functions or restructuring.

- ✓ DO: Direct, clear function names that describe what they do; small functions with few variables; early returns to flatten nesting
- ✗ DON'T: `AbstractFactoryImpl` patterns when simple functions suffice; functions that scroll for pages; pyramid-of-doom `if/else` chains

### 3. Readable

Optimise for human understanding. Code should be self-explanatory.

Imagine a new developer joining the team tomorrow. They're competent but unfamiliar with this codebase. Could they read your function and understand its purpose within a few minutes? Write for the newcomer. Your future self, returning to this code in six months, will thank you.

- ✓ DO: Clear, descriptive names that convey intent; code that reads like prose; `snake_case` for functions and variables, `PascalCase` for classes
- ✗ DON'T: Cryptic abbreviations; one-letter variables outside tight loops; code that requires tribal knowledge to understand

### 4. Explicit

Prefer clarity over implicit behaviour. Make data flow obvious. "Explicit is better than implicit."

- ✓ DO: Return values explicitly, raise specific exceptions, pass dependencies as arguments, use keyword-only arguments for non-obvious parameters
- ✗ DON'T: Hidden module-level state, monkey-patching, relying on import side effects, `from module import *`

### 5. Compositional

Use composition over inheritance. Build flexible systems with small classes, protocols, and plain functions.

- ✓ DO: Define `Protocol` classes for structural typing, prefer composition and dependency injection, use plain functions when no state is needed
- ✗ DON'T: Deep inheritance hierarchies; multiple inheritance for code reuse; abusing mixins as a substitute for proper design

### 6. Testable

Design for testing from the start. TDD drives better design.

- ✓ DO: Write the test first, inject dependencies through constructors or function arguments, use `pytest` fixtures, isolate I/O behind seams
- ✗ DON'T: Implement before test; reach for module-level singletons; bury network or filesystem calls deep inside business logic

### 7. Typed

Type hints are part of the contract, not optional decoration. They document intent and catch errors before runtime.

- ✓ DO: Annotate all public function signatures and class attributes; use `TypedDict`, `Protocol`, `Literal`, and generics where they clarify meaning; prefer `list[str]` / `dict[str, int]` (PEP 585) over `typing.List`
- ✗ DON'T: Sprinkle `Any` to silence the type checker; leave public APIs unannotated; use `# type: ignore` without a comment explaining why

### 8. Pragmatic

Solve the problem at hand without unnecessary complexity. We value simplicity over cleverness, and prefer obvious over optimal.

- ✓ DO: Start simple, refactor when patterns emerge; profile before optimising; write code that is immediately understandable
- ✗ DON'T: Over-engineer for hypothetical requirements; reach for metaclasses, decorators, or descriptors before simpler tools fail; write "clever" code that requires explanation to understand

When optimisation requires sacrificing clarity, accompany it with a comment explaining why the trade-off was made.

### 9. Predictable

Code should behave as expected without surprises. Handle errors explicitly.

- ✓ DO: Raise specific exception types, define custom exception hierarchies, document what each function raises, prefer pure functions and immutable values (`frozen=True` dataclasses, tuples)
- ✗ DON'T: Catch bare `Exception`; silently swallow errors with `except: pass`; rely on mutable default arguments; use exceptions for normal control flow

### 10. Statically Analysed

Use static analysis and linting to catch errors before runtime.

- ✓ DO: Run `mypy --strict` (or `pyright` in strict mode); use `ruff` for linting and formatting; enforce in CI
- ✗ DON'T: Ignore type errors with blanket `# type: ignore`; disable rules without justification; skip CI checks

### 11. Packaged

Package and distribute code in a way that is reproducible and standard.

- ✓ DO: Declare project metadata in `pyproject.toml` (PEP 621); pin dependencies with a lockfile (`uv.lock` / `poetry.lock`); isolate work in a virtual environment
- ✗ DON'T: Use `requirements.txt` without pinned versions; install into the system Python; rely on `setup.py` for new projects

### 12. Secure

Follow security best practices built into modern Python and frameworks.

- ✓ DO: Use parameterised queries or an ORM, validate input with `pydantic` or similar, use `secrets` for tokens, keep dependencies patched
- ✗ DON'T: Build SQL with f-strings; trust user input; use `pickle` on untrusted data; use `eval`/`exec` on input from outside the program

### 13. Well-Commented

Well-written code is self-evident about *what* it does. Comments add value when they explain *why*: the context, constraints, or reasoning that isn't obvious from the code itself.

- ✓ DO: Explain business rules, non-obvious constraints, or the reasoning behind a design choice; note when code works around an external limitation; use docstrings (PEP 257) for public modules, classes, and functions
- ✗ DON'T: Narrate what the code does step-by-step; leave comments that duplicate the function name; write docstrings that just restate the signature

---

### Framework Usage

When using frameworks (Django, FastAPI, Flask, etc.), follow their conventions while maintaining clean architecture boundaries.

- ✓ DO: Follow framework conventions, use built-in features (ORM, validation, DI), extend at designated extension points
- ✗ DON'T: Reinvent framework features; tightly couple domain logic to framework internals; bypass framework patterns for marginal convenience
