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

- `/advisor-opus:advise` — consult Opus for strategic advice at any point
- `/advisor-opus:plan` — get Opus to create a structured implementation plan (End State → Critical Path → Risks → First Move)
- `/advisor-opus:review` — have Opus review code changes, architecture, or approach with focus on correctness, edge cases, and security

### Smart Auto-Invocation

The plugin uses a **hybrid invocation strategy** designed to balance Opus call costs against value. Since each call spawns a full Opus sub-agent (significantly more expensive than the API's lightweight sub-inference), auto-invocation targets **2 calls per complex task, max 4**. All auto-calls gated to multi-step tasks (3+ steps, multiple files, or architectural decisions).

#### PLANNING (1 call — before execution)

One call before execution begins. Covers whichever moment comes **last** before you start writing code:

- **With plan mode**: plan mode exits (ExitPlanMode approved) → advisor reviews the plan before TaskCreate / implementation
- **Without plan mode**: after orientation (file reads, exploration) → advisor reviews before committing to an approach
- **Informal**: user confirms direction in conversation ("좋아 그렇게 하자", "let's go with that") → advisor reviews before proceeding

**Key rule: do NOT call twice** (once for orientation, again for plan confirmation). Pick whichever moment comes last — that's the one call.

```
      Orientation (file reads)
              │
      ┌───────┼────────────┐
      ▼                    ▼
  No plan mode         Plan mode
      │                    │
      │              ExitPlanMode
      │               approved
      └───────┬────────────┘
              ▼
     ★ PLANNING: 1 advisor call ★
              │
              ▼
      TaskCreate / implementation
```

#### COMPLETION (1 call — before done)

When a substantial unit of work is **code-complete and about to be committed, PR'd, or declared done** — all files written, tests run:

- Auto-calls the advisor for a final check on correctness, edge cases, and silent failures
- **Complements verification**: tests check "does it work", the advisor checks "did we miss anything"
- Make deliverables durable (save files) before this call — if the session ends during it, saved work persists

#### PIVOT (rare — change of approach)

Auto-suggested when the executor is about to **fundamentally change direction** mid-execution:

- Abandoning the current strategy for a different solution
- Reverting substantial work to try another way
- Switching architecture, library, or pattern mid-implementation

The advisor validates the new direction before you commit to it — catching cases where the original approach was actually correct, or where the new approach has non-obvious risks.

#### Reactive Calls (when stuck)

Auto-suggested when the executor detects it's stuck:

- Same error message or test failure **3+ times**
- Edit-fail loop on the **same file/function**
- **5+ tool calls** without meaningful progress

#### Always Skipped

Trivial tasks are never auto-escalated: single-file edits, typo fixes, simple additions, obvious next steps.

#### Manual Override

`/advise`, `/plan`, `/review` **always work** regardless of task complexity — the gates only apply to auto-invocation.

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
  │   Returns focused, actionable advice (~100 words)
  │
  └── You continue working, informed by Opus-level guidance
```

The advisor agent:
- **Reads** your codebase (via Read, Glob, Grep tools)
- **Advises** with specific recommendations, not vague suggestions
- **Does NOT modify** any files — advice only
- Returns concise, enumerated steps (~100 words default, expanded for plan/review tasks)

### How to Treat the Advice

Following the [official best practices](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool#best-practices):

- **Give the advice serious weight.** If a step fails empirically or you have primary-source evidence that contradicts a claim, adapt — but don't silently switch.
- **Reconcile conflicts explicitly.** If your data points one way and the advisor points another, surface the conflict in a follow-up call rather than guessing which is right.
- **Make deliverables durable before review calls.** Write files, save results, commit changes before calling `/review` — if the session ends during the advisor call, a saved result persists.

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

## Call Budget

Each Opus call spawns a full sub-agent — heavier than the API's lightweight sub-inference. The plugin targets **2 calls per complex task, max 4**:

```
Complex task lifecycle (3+ steps):

  ┌─ PLANNING ────────── 1 call (always)   before execution
  │
  │  ... implementation ...
  │
  │  ┌─ PIVOT ────────── 0-1 call (rare)   if changing direction
  │  │
  │  │  ┌─ REACTIVE ──── 0-1 call (rare)   if stuck
  │  │  │
  └──┴──┴─ COMPLETION ── 1 call (always)   before commit/PR/done
  ─────────────────────────────────────
  Typical: 2 calls    Max: 4 calls
```

Trivial tasks (single-file edits, typos, obvious next steps) are **always skipped** — zero auto-calls. Manual `/advise`, `/plan`, `/review` bypass all gates.

## Typical Flows

### Plan → Execute → Completion Review

```bash
# 1. Plan the work (PLANNING call fires automatically)
/advisor-opus:plan add rate limiting to the API endpoints

# 2. Execute the plan (Sonnet handles implementation)
# ... implement the feature ...

# 3. Code complete — COMPLETION call fires automatically before commit
#    → catches edge cases, silent failures before they enter git history
```

### Mid-Execution Pivot

```bash
# You're implementing approach A, realize it won't scale...
# → PIVOT auto-fires: "Approach B is better because X,
#    but watch out for Y when you switch."
```

### Auto-Escalation on Stuck

```bash
# Same error 3 times in a row...
# → REACTIVE auto-fires: "The root cause is X, not what you're looking at."
```

### Quick Decision Support

```bash
# Manual call — always works, no complexity gate
/advisor-opus:advise Redis vs. in-memory cache for session storage — we have 3 app instances behind a load balancer
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
| **Cost per call** | ~1,400-1,800 tokens (lightweight) | Full Opus agent (heavier) |
| **Cost control** | `max_uses`, prompt caching | Complexity gate (3+ steps), call budget (target 2, max 4) |
| **Auto-invocation** | Built-in timing guidance | 4 triggers: PLANNING + COMPLETION (typical) + PIVOT + REACTIVE (rare) |
| **For whom** | Developers building applications | Claude Code users during development |

This plugin is inspired by the Advisor Strategy pattern but implements it through Claude Code's native agent system, not through the API's `advisor_20260301` tool type. The **complexity gate** is unique to this plugin — a necessary adaptation since each call here is heavier than the API's sub-inference.

## References

- [The Advisor Strategy](https://claude.com/blog/the-advisor-strategy) — Anthropic blog post
- [Advisor Tool Documentation](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool) — API reference
- [Advisor Tool Best Practices](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool#best-practices) — Timing, prompting, and cost control

## License

MIT
