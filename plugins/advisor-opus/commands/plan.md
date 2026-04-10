---
description: Get Claude Opus to create a strategic implementation plan before starting complex work. Call after orientation (file reads, exploration) but BEFORE writing code or committing to an approach.
argument-hint: '[what to plan — feature, refactor, migration, etc.]'
context: fork
allowed-tools: Agent
---

Route this request to the `advisor-opus:opus-advisor` subagent with a planning-specific framing.

Raw user request:
$ARGUMENTS

## Routing Rules

1. If `$ARGUMENTS` is empty, ask the user what they want to plan.
2. Forward to the `advisor-opus:opus-advisor` subagent with this prefix:

   "Create a strategic implementation plan for the following task. This is a planning task — provide a comprehensive, detailed plan without the usual 100-word limit. Structure your response as:
   1. **End State** — what 'done' looks like
   2. **Critical Path** — ordered steps with dependencies
   3. **Risks** — what could go wrong and how to mitigate
   4. **First Move** — the single best action to start with

   Task: [user's request]"

3. Return the advisor's plan verbatim.

## How the Executor Should Treat the Plan

Give the plan serious weight. If you follow a step and it fails empirically, or you have primary-source evidence that contradicts a specific claim, adapt. But don't silently deviate — surface the conflict in a follow-up advisor call before committing to a different branch.
