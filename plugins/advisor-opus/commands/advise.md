---
description: Consult Claude Opus for strategic advice. Proactive auto-call only for complex multi-step tasks (3+ steps, multi-file, architecture decisions) — once before committing to an approach. Completion review — when a substantial work unit is code-complete and about to be committed, PR'd, or declared done, auto-call for final correctness/edge-case check. Key checkpoint — when an approach is decided (plan mode exit OR user confirms in conversation), auto-call before execution begins. Pivot auto-call when changing approach mid-execution. Reactive auto-call when stuck (same error 3+, edit-fail loop, 5+ tool calls without progress). SKIP for trivial work. Manual /advise always works.
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
3. Spawn the `advisor-opus:opus-advisor` subagent.
4. Return the advisor's response verbatim. Do not summarize, filter, or add commentary.
5. After returning the advice, do not take any further action — let the user decide how to proceed.

## How the Executor Should Treat the Advice

Give the advice serious weight. If you follow a step and it fails empirically, or you have primary-source evidence that contradicts a specific claim (the file says X, the paper states Y), adapt. A passing self-test is not evidence the advice is wrong — it's evidence your test doesn't check what the advice is checking.

If you've already retrieved data pointing one way and the advisor points another: don't silently switch. Surface the conflict in one more advisor call — "I found X, you suggest Y, which constraint breaks the tie?" The advisor saw your evidence but may have underweighted it; a reconcile call is cheaper than committing to the wrong branch.
