---
description: Consult Opus for strategic advice. Opus sessions — explicit /advise only. Sonnet/Haiku — auto-suggest at PLANNING (before execution), COMPLETION (before commit), PIVOT or REACTIVE (rare). Target 2 calls/task, max 4. Skip trivial work. Manual always works.
argument-hint: '[question or context for the advisor]'
context: fork
allowed-tools: Agent, Bash
---

Route this request to the `advisor-opus:opus-advisor` subagent.

**IMPORTANT**: The advisor agent starts fresh with NO conversation history. You (the executor) must build the context.

Raw user request:
$ARGUMENTS

## Context Gathering (executor does this BEFORE spawning the advisor)

The advisor starts with zero context — it cannot see your conversation at all. You must pass it everything it needs. Follow these three steps in order:

**Step 1 — Task summary** (1-2 sentences):
Write what the user is trying to do. Example: "User is adding OAuth2 login to a Next.js app."

**Step 2 — Key facts** (bullet list, keep short):
List what you know so far — files found, approaches tried, what worked/failed. Compress old history into short bullets.

**Step 3 — Recent raw output** (copy-paste, do NOT summarize):
Copy the last 3-5 tool results exactly as they appeared — file contents, grep results, error messages. The advisor needs the actual text, not your summary of it. If a tool returned 50 lines, include all 50 lines.

Optional: run `git diff --stat` and `git status` to show current state.

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
