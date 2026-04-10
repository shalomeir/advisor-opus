# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0] - 2026-04-10

### Changed

- **Breaking**: Rewrote agent and all commands to align with [official Advisor Tool best practices](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool#best-practices)
- Hybrid invocation strategy: proactive (complex tasks) + reactive (stuck detection) + key checkpoint (plan→execution)
- Complexity gate: auto-invocation only for multi-step tasks (3+ steps, multi-file, architecture decisions); trivial tasks always skipped
- Key checkpoint: auto-reviews approach at plan→execution transition — formal plan mode (ExitPlanMode) and informal conversation ("좋아 그렇게 하자")
- Output limit: 150 words → 100 words per official recommendation (~35-45% token reduction)
- All commands: added "How the Executor Should Treat the Advice" section (serious weight, conflict reconciliation via follow-up call)
- `review.md`: added durable deliverable reminder before advisor call
- Removed redundant `model: "opus"` references from command routing rules (declared in agent frontmatter)
- README: full rewrite reflecting v0.2.0 strategy, invocation flow diagrams, and best practices guidance

## [0.1.0] - 2026-04-10

### Added

- Initial release of advisor-opus plugin for Claude Code
- Three commands: `advise`, `plan`, `review`
- Single `opus-advisor` agent with Read, Glob, Grep tools
- Strategic advisor pattern: higher-intelligence model for planning, reviewing, and course-correcting

### Fixed

- `review.md`: replaced git diff reference with Glob/Read-based file exploration (agent has no Bash access)
- `plan.md` and `review.md`: added 150-word limit override for detailed output
- `plugin.json`: added missing `version` field to match `package.json`
