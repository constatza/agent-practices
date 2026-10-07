# agent-practices

Reusable, project-agnostic engineering practices for coding agents (Claude Code,
Codex, and anything else that reads `AGENTS.md`).

- **`AGENTS.md`** — language-agnostic practices: type safety, SOLID/YAGNI,
  testing hygiene, documentation, commit conventions. Claude Code (2.1.277+)
  reads this natively when a project has no `CLAUDE.md`; other agents that
  follow the [AGENTS.md convention](https://agents.md) read it directly too.
  Use it as a project starting point where appropriate; do not maintain it as
  a second user-level copy of the canonical shared instructions.
- **`plugins/python-practices`** — the Python-specific expression of those
  principles (tooling, typing, domain modeling, validation, testing, and
  documentation), packaged as an
  installable skill for both Claude Code and Codex.

## Why no `rust-practices` plugin here

[`leonardomso/rust-skills`](https://github.com/leonardomso/rust-skills) (MIT,
265 rules across 26 categories, multi-tool) already covers idiomatic Rust
comprehensively and is philosophically aligned with the principles in this
repo's `AGENTS.md`. Install it instead of duplicating that effort:

```bash
claude plugin marketplace add actionbook/rust-skills
claude plugin install rust-skills@rust-skills
```

## Companion: delegation policy

[`constatza/delegation-policy`](https://github.com/constatza/delegation-policy)
covers a different, complementary concern — bounding *when* and *how* an agent
delegates work to subagents — and isn't merged into this repo since it's a
coded policy engine with its own CI and release process, not static
docs/skills content. Install both if you want the full set.

## Install

Claude Code:
```bash
claude plugin marketplace add constatza/agent-practices
claude plugin install python-practices@agent-practices
```

Codex CLI:
```bash
codex plugin marketplace add constatza/agent-practices
codex plugin add python-practices@agent-practices
```

For a shared user-level configuration, keep the canonical instructions in
`$HOME/.claude/CLAUDE.md` and point `$HOME/.codex/AGENTS.md` to it with a
relative symlink (`../.claude/CLAUDE.md`). Project files may override those
defaults with project-specific rules.

## Maintaining

`plugins/python-practices/plugin.json` is the hand-edited source of truth for
the plugin's version; `.claude-plugin/plugin.json` and `.codex-plugin/
plugin.json` are the per-host manifests and should be kept in sync with it by
hand (no build step — this repo has no code, just docs and skill files).
