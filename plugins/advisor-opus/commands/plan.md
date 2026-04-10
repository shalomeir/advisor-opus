---
description: Get Opus to create an implementation plan. Opus sessions — explicit /plan only. Sonnet/Haiku — auto-suggest once before execution for complex tasks (3+ steps, multi-file). This IS the PLANNING call — don't also call /advise. Skip trivial work. Manual always works.
argument-hint: '[what to plan — feature, refactor, migration, etc.]'
context: fork
allowed-tools: Agent, Bash
---

Route this request to the `advisor-opus:opus-advisor` subagent with a planning-specific framing.

**IMPORTANT**: The advisor agent starts fresh with NO conversation history. You (the executor) must build the context.

Raw user request:
$ARGUMENTS

## Context Gathering (executor does this BEFORE spawning the advisor)

The advisor starts with zero context — it cannot see your conversation at all. You must pass it everything it needs. Follow these three steps in order:

**Step 1 — Task summary** (1-2 sentences):
Write what the user wants to build, change, or fix. Example: "User wants to migrate the REST API from Express to Hono."

**Step 2 — What you know** (bullet list, keep short):
List codebase structure, relevant files, technologies, constraints. Include key file paths. Compress old findings into short bullets.

**Step 3 — Recent raw output** (copy-paste, do NOT summarize):
Copy the last 3-5 tool results exactly as they appeared — file contents, grep results, exploration findings. The advisor needs the actual text to plan accurately. If a tool returned 50 lines, include all 50 lines.

Optional: run `git diff --stat` and `git status` to show current state.

Structure the advisor prompt as:

```
## Task
[1-2 sentence summary of what needs to be planned]

## Codebase Context
- [key files, structure, technologies]
- [constraints, dependencies discovered]
- [what has been explored so far]

## Recent Context (verbatim)
[last 3-5 tool calls/results — file reads, grep results, etc.]

## Planning Request
[user's $ARGUMENTS]
```

## Routing Rules

1. If `$ARGUMENTS` is empty, ask the user what they want to plan.
2. Build the context as described above.
3. Forward to the `advisor-opus:opus-advisor` subagent with the structured context, plus this prefix:

   "Create a strategic implementation plan for the following task. This is a planning task — provide a comprehensive, detailed plan without the usual 100-word limit. Structure your response as:
   1. **End State** — what 'done' looks like
   2. **Critical Path** — ordered steps with dependencies
   3. **Risks** — what could go wrong and how to mitigate
   4. **First Move** — the single best action to start with"

4. Return the advisor's plan verbatim.

## How the Executor Should Treat the Plan

Give the plan serious weight. If you follow a step and it fails empirically, or you have primary-source evidence that contradicts a specific claim, adapt. But don't silently deviate — surface the conflict in a follow-up advisor call before committing to a different branch.
