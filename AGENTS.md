# Core Practices

## Type Safety
- Use the strongest static typing the language provides on every public
  interface — function signatures, data structures, module boundaries. Treat a
  type error, from a compiler or a type checker, as a correctness bug, not a
  formality to silence.
- Model invariants directly in the type system: make illegal states
  unrepresentable rather than just checked at runtime. Avoid untyped escape
  hatches (`Any`, `void*`, stringly-typed values, dicts/maps standing in for
  records) where a real type would do.

## Architecture
- Prefer simplicity over complexity — YAGNI (You Aren't Gonna Need It): don't
  build abstractions, configuration, or flexibility for a need that doesn't
  exist yet. The simplest design that satisfies the current, real requirement
  is the right one until a second real use case actually appears.
- Before writing custom code for non-trivial functionality, check the standard
  library, adopted dependencies, and well-maintained third-party packages.
  Evaluate maintenance, licensing, security, compile/runtime cost, and
  transitive complexity; a small local implementation is preferable when a
  dependency's cost exceeds its value. Applies at every scale, from a single
  function to a whole subsystem.
- Don't Repeat Yourself (DRY): no duplicated logic across call sites.
- No magic values: a literal with meaning (a threshold, a limit, a status
  code, a path segment, a retry count) gets a named constant, not a bare
  number or string repeated at its call sites. Never hardcode a value that
  legitimately varies by environment, deployment, or caller — thread it
  through config or a parameter instead.
- SOLID, expressed through the language's native constructs (traits/protocols,
  closed enums, free functions) — not mechanical Java/C++-style class
  hierarchies or an interface for every single implementation:
  - **Single Responsibility** — a unit has one reason to change.
  - **Open/Closed** — open for extension, closed for modification (e.g. a
    registry/strategy pattern over a growing if/else chain of types).
  - **Liskov Substitution** — a subtype must be usable anywhere its supertype
    is expected, without surprising behavior.
  - **Interface Segregation** — depend on narrow, purpose-built interfaces,
    not broad ones with unused methods.
  - **Dependency Inversion** — depend on abstractions, not concrete
    implementations, across module boundaries.
- Single source of truth: never let the same fact — a doc, a config value, a
  dataset, a version number — live in two independently-maintained places.
  When a second copy is genuinely required (e.g. cross-tool compatibility),
  generate or derive it from the canonical source rather than hand-maintaining
  both; hand-maintained duplicates drift.
- Prefer composition over inheritance for orthogonal concerns.
- Use guard clauses / early returns, and exhaustive pattern matching
  (`match`/`switch` over a closed set of variants) instead of deeply nested
  `if`/`else` conditionals.
- Prefer immutable/frozen data structures and configs where practical.
- Keep functions and modules small and single-purpose — split a unit as soon as
  it has more than one reason to change.
- Favor a functional core, imperative shell: pure logic (no I/O, no side
  effects) stays independently testable; I/O and orchestration live in a thin
  outer layer.

## Testing
- Use reusable, composable fixtures for shared, complex, or expensive setup.
  Keep simple one-use test values inline and fixtures at the narrowest useful
  scope.
- Use the test framework's own isolated temp-path mechanism — never hand-roll
  temp file/directory logic.
- Tests run in strict isolation: no dependency on repo configs, external
  services, network access, or ambient machine/user state beyond what a
  fixture provides.
- Randomized logic must be deterministic in tests — seed explicitly.

## Documentation
- Explain the *why* directly in the doc/comment itself — never cite an
  external doc, plan, or ticket path as a substitute for explanation; those
  references rot as the codebase changes.
- Keep architecture/reference docs current — edit sections in place rather
  than appending deprecation notes or stale status remarks.

## Tooling
- Prefer `ast-grep` (structural/AST-aware search) over plain-text grep when
  searching code.

## Commit Messages
- Never mention Claude/AI authorship in commits.
- Git commit messages MUST follow Conventional Commits format
  (`type(scope): summary`) with a bulleted body explaining the changes — never
  a single unbulleted paragraph.

## Plans
- Follow the current project's planning convention. Create or update a plan
  document only when the project or user requires one, and never write project
  plans outside the project.

## Working with Files
- Never read raw PDF bytes/streams directly (e.g. piping a PDF through a web
  fetch tool, or regex/grep over raw PDF bytes for its *text content*) —
  convert with `markitdown` first (`uvx markitdown <file-or-url>` works
  without a pre-install) and read the resulting markdown/text. This is about
  extracting readable content; inspecting a PDF's actual binary structure
  (e.g. checking for link annotations) is a different task and not what this
  rule covers.
