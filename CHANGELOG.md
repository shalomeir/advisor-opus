# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
