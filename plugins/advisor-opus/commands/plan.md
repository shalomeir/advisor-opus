---
description: Get Claude Opus to create a strategic implementation plan. If you are Claude Opus, do NOT auto-invoke — only respond to explicit /plan commands. For Haiku/Sonnet: auto-suggest for complex multi-step tasks (3+ steps, multi-file, architecture decisions) once before execution begins — after plan mode exit or after orientation if no plan mode. This is the PLANNING call — do not also call /advise separately for the same decision point. SKIP for trivial or single-step work. Manual /plan always works regardless of executor model.
argument-hint: '[what to plan — feature, refactor, migration, etc.]'
context: fork
allowed-tools: Agent, Bash
---

Route this request to the `advisor-opus:opus-advisor` subagent with a planning-specific framing.

**IMPORTANT**: The advisor agent starts fresh with NO conversation history. You (the executor) must build the context.

Raw user request:
$ARGUMENTS

## Context Gathering (executor does this BEFORE spawning the advisor)

The advisor cannot see your conversation. You must summarize and forward it. Build the context in three layers:

1. **Task summary** (1-2 sentences): What is the user trying to build/change/fix?
2. **What you've learned so far** (bullet points, compressed): Codebase structure, relevant files found, constraints discovered, technologies in use. Include key file paths. Keep earlier findings compressed to essentials.
3. **Recent context** (raw, uncompressed): The last 3-5 tool calls and their results — file reads, grep results, exploration findings. Include these **verbatim**. The advisor needs to see exactly what you've discovered to plan accurately.

Optionally, run `git diff --stat` and `git status` to show current state.

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
