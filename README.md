# Advisor Opus — Claude Code Plugin

Bring the [Advisor Strategy](https://claude.com/blog/the-advisor-strategy) into Claude Code. Consult Claude Opus as a strategic advisor mid-task — without switching your entire session to Opus.

> **This is an unofficial, community-driven plugin.** It is NOT an official Anthropic product and is not affiliated with or endorsed by Anthropic.

## Why the Advisor Strategy?

Anthropic introduced the [Advisor Tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool) as an API-level pattern: pair a fast executor model (Sonnet/Haiku) with a higher-intelligence advisor (Opus) that provides strategic guidance mid-generation. The results are compelling:

| Configuration | Intelligence | Cost |
|---|---|---|
| Sonnet solo | Baseline | Baseline |
| **Sonnet + Opus advisor** | **+2.7pp on SWE-bench** | **11.9% cheaper per task** |
| Haiku + Opus advisor | 2x BrowseComp score | 85% cheaper than Sonnet |

The key insight: **most work is mechanical, but having an excellent plan is crucial.** The executor handles the bulk of token generation at lower rates while Opus steps in only at critical decision points.

### From API to Claude Code

The official Advisor Tool works at the API level — you integrate it into your own applications via the Messages API. **This plugin brings the same pattern into Claude Code itself**, so you can consult Opus without leaving your terminal session:

- Running Claude Code with Sonnet? Call `/advisor-opus:advise` to get Opus-level guidance on demand.
- About to start complex work? Use `/advisor-opus:plan` for an Opus-crafted implementation plan.
- Finished coding? Run `/advisor-opus:review` for an Opus-level review of your changes.

No API integration needed. No code changes. Just a slash command — or let Claude decide for you.

## What You Get

### Slash Commands (Explicit)

- `/advisor-opus:advise` — consult Opus for strategic advice when stuck, facing a key decision, or before committing to an approach
- `/advisor-opus:plan` — get Opus to create a structured implementation plan (End State → Critical Path → Risks → First Move)
- `/advisor-opus:review` — have Opus review code changes, architecture, or approach with focus on correctness, edge cases, and security

### Auto-Invocation (Proactive)

The `opus-advisor` agent is also registered as a subagent that Claude Code can spawn on its own when the situation calls for it. In these cases, Claude Code may **automatically consult Opus** without an explicit slash command:

- **Stuck or going in circles** — repeated failures, errors not converging, approach not working
- **Complex planning needed** — multi-step task that requires a strategy before diving in
- **Architecture decisions** — choosing between patterns, technologies, or trade-offs
- **Debugging dead ends** — when the current model needs a higher-intelligence perspective

Auto-invocation depends on Claude Code's own judgment about when to escalate. For **guaranteed** Opus consultation, use the slash commands directly.

## How It Works

```
You (Claude Code session, any model)
  │
  ├── /advisor-opus:advise "should I use WebSocket or SSE here?"
  │       │
  │       ▼
  │   Opus subagent spawns (reads your full context)
  │       │
  │       ▼
  │   Returns focused, actionable advice (~150 words)
  │
  └── You continue working, informed by Opus-level guidance
```

The advisor agent:
- **Reads** your codebase (via Read, Glob, Grep tools)
- **Advises** with specific recommendations, not vague suggestions
- **Does NOT modify** any files — advice only
- Returns concise, enumerated steps following the official Advisor Strategy prompting patterns

## Requirements

- **Claude Code** — the [Anthropic CLI](https://claude.ai/code)
- **Claude Opus access** — the advisor subagent runs on Claude Opus 4.6

## Install

In Claude Code, run:

```bash
/plugin marketplace add shalomeir/advisor-opus
/plugin install advisor-opus@advisor-opus
/reload-plugins
```

After install, verify the commands are available:

```bash
/advisor-opus:advise hello
```

You should see the Opus advisor respond with guidance.

### Update

```bash
/plugin marketplace update advisor-opus
/reload-plugins
```

### Local Development

```bash
git clone https://github.com/shalomeir/advisor-opus.git
claude --plugin-dir ./advisor-opus
```

## Usage

### `/advisor-opus:advise`

General-purpose strategic consultation. Use at any point when you want Opus-level thinking.

**When to use:**
- Before committing to an approach on a complex task
- When stuck on a bug and need a fresh perspective
- When facing an architectural decision with multiple valid options
- When something feels off but you can't pinpoint why

**Examples:**

```bash
# General advice on current work
/advisor-opus:advise

# Specific question
/advisor-opus:advise should I use a queue or a cron job for this batch processing?

# Decision support
/advisor-opus:advise I'm torn between adding a cache layer vs. optimizing the DB query — which approach has fewer risks?

# Debug guidance
/advisor-opus:advise the tests pass locally but fail in CI — what should I investigate first?
```

### `/advisor-opus:plan`

Creates a structured implementation plan before you start complex work.

**When to use:**
- Before starting a multi-file feature implementation
- Before a refactoring that touches many modules
- When planning a migration or upgrade
- When you need to break down a large task into safe steps

**Examples:**

```bash
# Feature planning
/advisor-opus:plan add OAuth2 authentication with Google and GitHub providers

# Refactoring
/advisor-opus:plan migrate the REST API from Express to Hono

# Migration
/advisor-opus:plan upgrade from Next.js 14 to 16 with App Router

# Architecture
/advisor-opus:plan design the data layer for a multi-tenant SaaS
```

The plan follows a structured format:
1. **End State** — what "done" looks like
2. **Critical Path** — ordered steps with dependencies
3. **Risks** — what could go wrong and how to mitigate
4. **First Move** — the single best action to start with

### `/advisor-opus:review`

Opus-powered review of your code changes, architecture, or implementation approach.

**When to use:**
- Before creating a PR
- After implementing a complex feature
- When you want a second opinion on your approach
- When reviewing security-sensitive code

**Examples:**

```bash
# Review current changes
/advisor-opus:review

# Review specific files
/advisor-opus:review the authentication middleware in src/middleware/auth.ts

# Architecture review
/advisor-opus:review the database schema design for the new billing system

# Security review
/advisor-opus:review check the input validation and sanitization in the API layer
```

The review focuses on:
- **Correctness** — bugs, logic errors, unhandled edge cases
- **Architecture** — design flaws, coupling, scalability concerns
- **Security** — vulnerabilities, data exposure, injection risks
- **Silent Failures** — swallowed errors, missing validation, unsafe defaults

## Typical Flows

### Plan → Execute → Review

```bash
# 1. Get Opus to plan the work
/advisor-opus:plan add rate limiting to the API endpoints

# 2. Execute the plan (you or Claude Code with Sonnet)
# ... implement the feature ...

# 3. Have Opus review the result
/advisor-opus:review
```

### Quick Decision Support

```bash
# Ask for advice before a key decision
/advisor-opus:advise Redis vs. in-memory cache for session storage — we have 3 app instances behind a load balancer
```

### Debug Consultation

```bash
# When stuck, get Opus to analyze the situation
/advisor-opus:advise I've tried X and Y but the memory leak persists — what am I missing?
```

## Architecture

This plugin is intentionally simple — no companion scripts, no external CLIs, no background job management.

```
User ─── /advisor-opus:* slash command
           │
           ▼
   Command file (commands/*.md)
   Frames the request for the advisor's specific mode
           │
           ▼
   opus-advisor agent (agents/opus-advisor.md)
   Spawned as a subagent with model: opus
   Reads codebase via Read/Glob/Grep, returns advice
```

The entire plugin is three command files and one agent definition — easy to understand, customize, and extend.

## Differences from the Official Advisor Tool API

| | Advisor Tool API | This Plugin |
|---|---|---|
| **Where** | Messages API (`advisor_20260301`) | Claude Code CLI |
| **How** | Server-side sub-inference within a single API request | Subagent spawned with `model: opus` |
| **For whom** | Developers building applications | Claude Code users during development |
| **Billing** | Per-token at Opus rates for advisor turns | Part of your Claude Code subscription/usage |
| **Control** | `max_uses`, prompt caching, streaming pause | On-demand via slash commands |

This plugin is inspired by the Advisor Strategy pattern but implements it through Claude Code's native agent system, not through the API's `advisor_20260301` tool type.

## References

- [The Advisor Strategy](https://claude.com/blog/the-advisor-strategy) — Anthropic blog post
- [Advisor Tool Documentation](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool) — API reference

## License

MIT
