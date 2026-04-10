---
description: Consult Claude Opus for strategic advice on the current task. Use when stuck, facing a key decision, or before committing to an approach.
argument-hint: '[question or context for the advisor]'
context: fork
allowed-tools: Agent
---

Route this request to the `advisor-opus:opus-advisor` subagent.

The advisor sees the full conversation context automatically. Your job is to forward the request and return the advice verbatim.

Raw user request:
$ARGUMENTS

## Routing Rules

1. If `$ARGUMENTS` is empty, forward the current conversation context with: "Review the current task and conversation. Provide strategic guidance on the best approach, flag any risks, and suggest next steps."
2. If `$ARGUMENTS` is provided, forward it as-is to the advisor.
3. Spawn the `advisor-opus:opus-advisor` subagent with `model: "opus"`.
4. Return the advisor's response verbatim. Do not summarize, filter, or add commentary.
5. After returning the advice, do not take any further action — let the user decide how to proceed.
