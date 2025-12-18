# Code Quality

## Overview

This document covers code quality tools, conventions, and automation for maintaining a clean, consistent codebase.

## Tool Stack

| Tool | Purpose |
|------|---------|
| **Ruff** | Linting + formatting (replaces Black, isort, Flake8) |
| **MyPy** | Static type checking |
| **Pre-commit** | Git hooks for automated checks |
| **Slotscheck** | Verify `__slots__` usage |

## Ruff Configuration

```toml
# pyproject.toml
[tool.ruff]
target-version = "py313"
line-length = 88
fix = true

[tool.ruff.lint]
select = [
    "E",     # pycodestyle errors
    "W",     # pycodestyle warnings
    "F",     # Pyflakes
    "I",     # isort
    "B",     # flake8-bugbear
    "C4",    # flake8-comprehensions
    "UP",    # pyupgrade
    "ARG",   # flake8-unused-arguments
    "SIM",   # flake8-simplify
    "TCH",   # flake8-type-checking
    "PTH",   # flake8-use-pathlib
    "ERA",   # eradicate
    "PL",    # Pylint
    "RUF",   # Ruff-specific rules
]
ignore = [
    "E501",   # line too long (handled by formatter)
    "PLR0913", # too many arguments
    "PLR2004", # magic value comparison
]

[tool.ruff.lint.isort]
known-first-party = ["app"]
force-single-line = false
lines-after-imports = 2

[tool.ruff.lint.per-file-ignores]
"tests/*" = ["ARG", "PLR2004"]
"**/alembic/*" = ["ERA"]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
```

## MyPy Configuration

```toml
# pyproject.toml
[tool.mypy]
python_version = "3.13"
strict = true
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
no_implicit_optional = true
warn_redundant_casts = true
warn_unused_ignores = true
show_error_codes = true

plugins = [
    "pydantic.mypy",
    "sqlalchemy.ext.mypy.plugin",
]

[[tool.mypy.overrides]]
module = [
    "celery.*",
    "uvicorn.*",
]
ignore_missing_imports = true
```

## Pre-commit Configuration

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.8.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.13.0
    hooks:
      - id: mypy
        additional_dependencies:
          - pydantic>=2.0
          - sqlalchemy[mypy]>=2.0
        args: [--config-file, pyproject.toml]

  - repo: https://github.com/ariebovenberg/slotscheck
    rev: v0.19.0
    hooks:
      - id: slotscheck
        files: ^src/

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
      - id: check-merge-conflict
      - id: debug-statements

default_language_version:
  python: python3.13
```

## Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Files | `snake_case.py` | `user_repository.py` |
| Classes | `PascalCase` | `UserRepository` |
| Functions/Methods | `snake_case` | `get_by_id` |
| Constants | `UPPER_SNAKE` | `MAX_CONNECTIONS` |
| Private | `_prefix` | `_internal_method` |
| Type Vars | `T`, `KeyT` | `T = TypeVar("T")` |

## Code Style Examples

### Imports

```python
# Good - grouped and sorted
from __future__ import annotations

import asyncio
from dataclasses import dataclass
from typing import TYPE_CHECKING
from uuid import UUID

from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from app.domain.entities.user import User
from app.domain.value_objects.user_id import UserId

if TYPE_CHECKING:
    from app.application.common.ports.unit_of_work import UnitOfWork
```

### Type Hints

```python
# Use modern Python 3.10+ syntax
def get_users(ids: list[UUID]) -> list[User]: ...

# Union types
def get_user(user_id: UUID) -> User | None: ...

# Generic classes
class Repository[T]:
    def add(self, entity: T) -> None: ...
```

### Docstrings

```python
def create_user(
    username: str,
    password: str,
    role: UserRole = UserRole.USER,
) -> User:
    """
    Create a new user with the given credentials.
    
    Args:
        username: Unique username for the user.
        password: Raw password (will be hashed).
        role: User role, defaults to USER.
    
    Returns:
        Newly created User entity.
    
    Raises:
        InvalidUsernameError: If username doesn't meet requirements.
        InvalidPasswordError: If password is too weak.
    """
```

## Makefile Commands

```makefile
# Makefile
.PHONY: lint format type-check test

lint:
	ruff check src tests

format:
	ruff format src tests
	ruff check --fix src tests

type-check:
	mypy src

test:
	pytest

check: lint type-check test
	@echo "All checks passed!"

pre-commit:
	pre-commit run --all-files
```

## VS Code Settings

```json
// .vscode/settings.json
{
    "python.defaultInterpreterPath": ".venv/bin/python",
    "[python]": {
        "editor.formatOnSave": true,
        "editor.defaultFormatter": "charliermarsh.ruff",
        "editor.codeActionsOnSave": {
            "source.fixAll.ruff": "explicit",
            "source.organizeImports.ruff": "explicit"
        }
    },
    "python.analysis.typeCheckingMode": "strict",
    "mypy-type-checker.args": ["--config-file=pyproject.toml"]
}
```

---

**Previous**: [Testing](13-testing.md) | **Next**: [Performance](15-performance.md)
