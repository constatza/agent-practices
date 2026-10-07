---
name: python-practices
description: Python engineering conventions for tooling, typing, type-driven domain modeling, validation, testing, and docstrings. Use when writing, reviewing, or refactoring Python code.
---

# Python Practices

Apply these Python-specific rules alongside the active language-agnostic and project instructions. Project requirements take precedence when they select a different supported Python version or toolchain.

## Tooling

- Run Python entry points through `uv run ...` for reproducible tooling.
- Use `ruff` for linting and safe autofixes.
- Run the project's configured static type checker in CI with strict settings where supported. `ty`, Pyright, mypy, and other maintained checkers are all valid choices; select based on the project, and do not rule out `ty` by default.

## Typing

- Use PEP 695 generic syntax (`class Foo[T]:`, `def foo[T]():`) when the supported Python version permits it; use legacy `TypeVar`/`Generic[T]` only when compatibility requires it.
- Minimize `Any`, broad `cast(...)`, and `# type: ignore`. Each escape hatch requires a concrete justification and the narrowest possible scope.
- Do not imitate Rust ownership semantics mechanically; use Python's native object and resource-management model.

## Type-Driven Domain Modeling

When creating or changing domain states, lifecycle APIs, validated values, result models, or boundary schemas, read [references/type-driven-domain-modeling.md](references/type-driven-domain-modeling.md).

Use these defaults:

- Represent mutually exclusive states with `A | B` unions instead of flags coupled to `None`.
- Model states with different data or operations as separate `@dataclass(frozen=True, slots=True)` classes when appropriate. Remember that `frozen=True` prevents attribute rebinding but does not make mutable field values deeply immutable.
- Use `NewType` for zero-cost semantic distinctions that need no runtime validation.
- Use immutable value objects when validation or behavior belongs to the value.
- Use `StrEnum` for closed symbolic values, but not as a substitute for state-specific types carrying different data.
- Handle closed unions with `match` and `typing.assert_never` when exhaustive checking matters.
- Use the project's existing validation mechanism at external boundaries. When complex structured input needs schema validation and the project uses or can justify Pydantic, prefer Pydantic v2, enable strict validation where coercion would hide invalid input, and convert boundary models into domain types rather than making them the domain by default.
- Keep `dict[str, Any]` and other weak representations out of core domain logic.

## Error Handling and Outcomes

- Raise exceptions for error transport. Let exceptions propagate until a caller can handle them meaningfully.
- Use explicit success/failure unions when success and failure are domain outcomes represented as data; do not introduce result wrappers merely to transport Python errors.

## Testing

- Use modular, composable pytest fixtures for shared, complex, or expensive setup. Keep simple one-use values inline, define fixtures at the narrowest useful scope, and place them in `conftest.py` only when they are shared across test modules.
- Use pytest's `tmp_path` fixture directly; do not use `tempfile`.
- Compare paths with `.as_posix()` when tests require a platform-independent textual path.

## Documentation

- Use Google-style docstrings.
- Describe semantics, constraints, errors, and non-obvious behavior without repeating types already expressed by annotations. Include type details only when annotations cannot express a relevant constraint.
