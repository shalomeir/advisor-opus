---
description: Consult Claude Opus for strategic advice. If you are Claude Opus, do NOT auto-invoke — only respond to explicit /advise commands. For Haiku/Sonnet: aim for 2 calls per complex task, max 4. PLANNING (1 call) — for multi-step tasks (3+ steps, multi-file, architecture), call once before execution begins. COMPLETION (1 call) — when work is code-complete and about to be committed/PR'd/declared done. PIVOT (rare) — when changing approach mid-execution. REACTIVE (rare) — when stuck (same error 3+, edit-fail loop, 5+ tool calls). SKIP for trivial work. Manual /advise always works regardless of executor model.
argument-hint: '[question or context for the advisor]'
context: fork
allowed-tools: Agent, Bash
---

Route this request to the `advisor-opus:opus-advisor` subagent.

**IMPORTANT**: The advisor agent starts fresh with NO conversation history. You (the executor) must build the context.

Raw user request:
$ARGUMENTS

## Context Gathering (executor does this BEFORE spawning the advisor)

The advisor cannot see your conversation. You must summarize and forward it. Build the context in three layers:

1. **Task summary** (1-2 sentences): What is the overall goal of this session? What did the user ask for?
2. **Key decisions and findings** (bullet points, compressed): What approaches were considered or tried? What worked, what failed? What was decided? Keep this concise — compress earlier history into key facts.
3. **Recent context** (raw, uncompressed): The last 3-5 tool calls, their results, any recent errors, test outputs, or user messages — include these **verbatim**. This is the most critical part. The advisor needs to see exactly what just happened to give relevant advice.

Optionally, run `git diff --stat` and `git status` to include current working state.

Structure the advisor prompt as:

```
## Task
[1-2 sentence summary of the session goal]

## Background
- [compressed key decisions and findings]
- [what was tried, what failed]
- [current approach and status]

## Recent Context (verbatim)
[last 3-5 tool calls/results, errors, user messages — raw, uncompressed]

## Question
[user's $ARGUMENTS, or "What should I do next?"]
```

## Routing Rules

1. Build the context as described above.
2. If `$ARGUMENTS` is provided, include it as the `## Question` section.
3. If `$ARGUMENTS` is empty, set the question to: "Review the current situation. Provide strategic guidance on the best approach, flag any risks, and suggest next steps."
4. Spawn the `advisor-opus:opus-advisor` subagent with the full structured prompt.
5. Return the advisor's response verbatim. Do not summarize, filter, or add commentary.
6. After returning the advice, do not take any further action — let the user decide how to proceed.

## How the Executor Should Treat the Advice

Give the advice serious weight. If you follow a step and it fails empirically, or you have primary-source evidence that contradicts a specific claim (the file says X, the paper states Y), adapt. A passing self-test is not evidence the advice is wrong — it's evidence your test doesn't check what the advice is checking.

If you've already retrieved data pointing one way and the advisor points another: don't silently switch. Surface the conflict in one more advisor call — "I found X, you suggest Y, which constraint breaks the tie?" The advisor saw your evidence but may have underweighted it; a reconcile call is cheaper than committing to the wrong branch.
