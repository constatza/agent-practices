---
name: python-practices
description: Python-specific engineering conventions — tooling, typing, testing, docstrings. Use when writing, reviewing, or refactoring Python code.
---

# Python Practices

The Python-language expression of the generic engineering practices in this
repo's `AGENTS.md`. Read that first — this skill only covers what's specific to
Python; it doesn't restate the language-agnostic principles (SOLID, YAGNI,
testing isolation, commit format, etc.).

## Tooling
- Launch all Python entry points through `uv run ...` for consistent,
  reproducible tooling.
- Use `ruff` for linting and autofix.
- Use `ty` for type checking — never `pyright`.
- Prefer `ast-grep` over plain-text grep when searching Python code.

## Typing
- Use PEP 695 generic syntax (`class Foo[T]:`, `def foo[T]():`) — never
  legacy `TypeVar`/`Generic[T]`.
- Use Pydantic models for structured/validated data and config, not raw dicts.
  This is the Python expression of the global Type Safety principle: make
  illegal states unrepresentable via Pydantic validators/`Literal`/enums, not
  just annotations that nothing enforces at runtime.

## Error Handling
- Raise exceptions directly — no error/result-wrapper objects for error
  transport. Exceptions propagate; callers catch what they can handle.

## Testing
- pytest fixtures in `conftest.py` — modular and composable, never inline test
  data (the Python expression of the global fixtures-only testing rule).
- Use pytest's `tmp_path` fixture directly (it's a ready-to-use
  `pathlib.Path`) — never the `tempfile` package.
- Use `.as_posix()` when comparing paths in tests, for cross-platform
  stability.

## Documentation
- Google-style docstrings, with type information included in the docstring's
  parameter/return descriptions in addition to signature type hints.
  **Note:** this duplicates information the type checker already enforces
  from the signature, and can drift out of sync with it — some Google-style
  Python codebases now omit repeating types in the `Args:` block for exactly
  that reason. Kept here as the current convention; reconsider if drift
  becomes a recurring problem.
