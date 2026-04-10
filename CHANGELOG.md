# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0] - 2026-04-10

### Changed

- **Breaking**: Rewrote agent and all commands to align with official Advisor Tool best practices
- Agent description: replaced loop-detection triggers with official timing guidance (call before substantive work, when stuck, before declaring done)
- Output limit: 150 words → 100 words per official recommendation (~35-45% token reduction)
- All commands: added "How the Executor Should Treat the Advice" section (serious weight, conflict reconciliation via follow-up call)
- `plan.md`: timing guidance — call after orientation, before writing code
- `review.md`: timing guidance — call after file writes/tests, before declaring done; added durable deliverable reminder
- `advise.md`: removed loop-detection framing, aligned with official call-timing patterns
- Removed redundant `model: "opus"` references from command routing rules (declared in agent frontmatter)

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
