---
description: Get Claude Opus to create a strategic implementation plan before starting complex work. Produces a step-by-step plan with risk analysis and critical path.
argument-hint: '[what to plan — feature, refactor, migration, etc.]'
context: fork
allowed-tools: Agent
---

Route this request to the `advisor-opus:opus-advisor` subagent with a planning-specific framing.

Raw user request:
$ARGUMENTS

## Routing Rules

1. If `$ARGUMENTS` is empty, ask the user what they want to plan.
2. Forward to the `advisor-opus:opus-advisor` subagent with `model: "opus"` and this prefix:

   "Create a strategic implementation plan for the following task. Structure your response as:
   1. **End State** — what 'done' looks like
   2. **Critical Path** — ordered steps with dependencies
   3. **Risks** — what could go wrong and how to mitigate
   4. **First Move** — the single best action to start with

   Task: [user's request]"

3. **Important**: Tell the advisor that this is a planning task — it should provide a comprehensive, detailed plan without the usual 150-word limit. Use structured sections for clarity.
4. Return the advisor's plan verbatim.
