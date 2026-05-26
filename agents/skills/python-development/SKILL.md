---
name: python-development
description: >
  Python development patterns for Python 3.12+ code. Use when: (1) Working on Python projects,
  (2) Writing new Python code, (3) Reviewing Python code, (4) Refactoring Python code,
  (5) Configuring mypy, ruff, or dependency tooling (uv/poetry). Covers idiomatic, typed,
  testable Python that applies to any modern Python codebase.
---

# Python Development

Core principles for writing clean, maintainable, and idiomatic Python 3.12+ code.

## Philosophy

### 1. Pythonic
Write Python the way Python is meant to be written. Follow PEP 8 and PEP 20.
- DO: Use comprehensions, context managers, dataclasses, `pathlib`, f-strings
- DON'T: Force Java/C++ idioms (manual getters/setters, index-based loops, `os.path` strings)

### 2. Concise
Keep code minimal without sacrificing clarity.
- DO: Small functions with few variables; early returns to flatten nesting
- DON'T: Functions that scroll for pages; pyramid `if/else` chains

### 3. Readable
Optimise for human understanding.
- DO: Descriptive names, `snake_case` functions, code that reads like prose
- DON'T: Cryptic abbreviations; clever one-liners that obscure intent

### 4. Explicit
Prefer clarity over implicit behaviour.
- DO: Return values explicitly, pass dependencies as arguments, raise specific exceptions
- DON'T: Hidden module state, import side effects, `from module import *`

### 5. Compositional
Build with composition, protocols, and small classes — not deep inheritance.
- DO: `Protocol` for structural typing, dependency injection, plain functions when stateless
- DON'T: Deep inheritance trees, mixin soup, multiple inheritance for code reuse

### 6. Testable
Design for testability from the start.
- DO: Write the test first, inject dependencies, isolate I/O behind seams
- DON'T: Module-level singletons, network/filesystem calls buried in domain logic

### 7. Typed
Type hints are part of the contract.
- DO: Annotate public signatures; use `Protocol`, `TypedDict`, `Literal`, generics
- DON'T: Sprinkle `Any`; leave public APIs unannotated; unexplained `# type: ignore`

### 8. Pragmatic
Solve the problem without unnecessary complexity.
- DO: Start simple, profile before optimising
- DON'T: Reach for metaclasses or decorators before simpler tools fail

### 9. Predictable
Handle errors explicitly. No surprises.
- DO: Raise specific exceptions, prefer pure functions and immutable values
- DON'T: Bare `except`; mutable default arguments; exceptions for control flow

### 10. Statically Analysed
Catch errors before runtime.
- DO: `mypy --strict` (or `pyright`), `ruff` for lint and format, enforced in CI
- DON'T: Blanket `# type: ignore`; disabled rules without justification

### 11. Packaged
Reproducible, standard packaging.
- DO: `pyproject.toml` (PEP 621), lockfile-based dependency manager, virtual environments
- DON'T: Unpinned `requirements.txt`; system Python; legacy `setup.py` for new projects

### 12. Secure
Follow built-in security best practices.
- DO: Parameterised queries, validate input with `pydantic`, `secrets` for tokens
- DON'T: SQL from f-strings; `pickle` untrusted data; `eval`/`exec` on input

## Error Handling

Quick reference:

```python
from __future__ import annotations


class UserNotFoundError(LookupError):
    """Raised when a user lookup fails."""

    def __init__(self, user_id: str) -> None:
        super().__init__(f"User not found: {user_id}")
        self.user_id = user_id


def get_user(user_id: str, *, repository: UserRepository) -> User:
    user = repository.find(user_id)
    if user is None:
        raise UserNotFoundError(user_id)
    return user


# Catch specifically. Chain with `from` to preserve context.
try:
    user = get_user(user_id, repository=repo)
except UserNotFoundError as exc:
    raise HTTPNotFound(str(exc)) from exc
```

## Testing

Use `pytest`. Parametrise data-driven cases. Keep arrange/act/assert sections visible.

```python
import pytest

from app.slugger import slug


@pytest.mark.parametrize(
    ("raw", "expected"),
    [
        pytest.param("Hello World", "hello-world", id="lowercase"),
        pytest.param("Héllo Wörld!", "hello-world", id="special-characters"),
        pytest.param("  spaced  ", "spaced", id="trims-whitespace"),
    ],
)
def test_slug(raw: str, expected: str) -> None:
    assert slug(raw) == expected


@pytest.fixture
def repository(tmp_path) -> UserRepository:
    return UserRepository(path=tmp_path / "users.db")


def test_get_user_raises_when_missing(repository: UserRepository) -> None:
    with pytest.raises(UserNotFoundError, match="missing-id"):
        get_user("missing-id", repository=repository)
```

## Type Hints & Static Analysis

Annotate public signatures. Prefer built-in generics (PEP 585) and `Protocol` for structural contracts.

```python
from __future__ import annotations

from collections.abc import Iterable
from dataclasses import dataclass
from typing import Protocol, TypedDict


class UserRow(TypedDict):
    id: int
    name: str
    roles: list[str]


@dataclass(frozen=True, slots=True)
class User:
    id: int
    name: str
    roles: tuple[str, ...]


class UserRepository(Protocol):
    def find(self, user_id: str) -> User | None: ...
    def save(self, user: User) -> None: ...


def active_admins(users: Iterable[User]) -> list[User]:
    return [u for u in users if "admin" in u.roles]
```

Configure `mypy` strictly in `pyproject.toml`:

```toml
[tool.mypy]
python_version = "3.12"
strict = true
warn_unused_ignores = true
warn_redundant_casts = true
disallow_any_explicit = false  # allow Any when explicit and justified
```

Configure `ruff` for lint + format:

```toml
[tool.ruff]
target-version = "py312"
line-length = 100

[tool.ruff.lint]
select = ["E", "F", "I", "B", "UP", "SIM", "RUF", "N", "PL"]
ignore = []

[tool.ruff.format]
quote-style = "double"
```

## Dependency Management

Prefer `uv` (fast, modern, Rust-based). `poetry` is an acceptable alternative. Both produce a lockfile — commit it for applications.

`pyproject.toml` with `uv`:

```toml
[project]
name = "my-app"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "httpx>=0.27,<1.0",
    "pydantic>=2.7,<3.0",
]

[dependency-groups]
dev = [
    "pytest>=8.0",
    "mypy>=1.10",
    "ruff>=0.5",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

Workflow:

```bash
# Create venv and install locked dependencies
uv sync

# Add a runtime dependency (updates pyproject.toml and uv.lock)
uv add httpx

# Add a dev dependency
uv add --dev pytest

# Run a tool inside the project venv
uv run pytest
uv run mypy src
uv run ruff check .
```

Rules:

- Commit `uv.lock` (or `poetry.lock`) for applications. Libraries pin only minimum compatible versions in `pyproject.toml` and do not commit a lockfile.
- Never `pip install` into the system Python. Always work inside a project-managed virtual environment.
- Pin Python version with `requires-python` and a `.python-version` file so contributors and CI agree.

## Dependency Injection

Pass dependencies as constructor or function arguments. Define `Protocol` interfaces at the boundary.

```python
from __future__ import annotations

import logging
from typing import Protocol


class PaymentGateway(Protocol):
    def charge(self, amount_cents: int) -> str: ...
    def refund(self, transaction_id: str) -> None: ...


class OrderRepository(Protocol):
    def save(self, order: Order) -> None: ...


class OrderService:
    def __init__(
        self,
        *,
        orders: OrderRepository,
        payments: PaymentGateway,
        logger: logging.Logger,
    ) -> None:
        self._orders = orders
        self._payments = payments
        self._logger = logger

    def place_order(self, order: Order) -> None:
        transaction_id = self._payments.charge(order.total_cents)
        self._orders.save(order)
        self._logger.info("order placed", extra={"order_id": order.id, "txn": transaction_id})
```

Avoid the service-locator anti-pattern (passing a container around). Inject the concrete dependencies you need so they are visible in the signature.
