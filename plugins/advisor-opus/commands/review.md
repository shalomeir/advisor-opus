---
description: Have Claude Opus review code changes, architecture decisions, or implementation approach. If you are Claude Opus, do NOT auto-invoke — only respond to explicit /review commands. For Haiku/Sonnet: auto-suggest for complex multi-step tasks (3+ steps, multi-file changes) when work is code-complete — all files written, tests run, about to commit or declare done. Complements verification (tests check "does it work", advisor checks "did we miss anything"). SKIP for trivial changes. Manual /review always works regardless of executor model.
argument-hint: '[what to review — files, approach, architecture, etc.]'
context: fork
allowed-tools: Agent, Bash
---

Route this request to the `advisor-opus:opus-advisor` subagent with a review-specific framing.

**IMPORTANT**: The advisor agent starts fresh with NO conversation history. You (the executor) must build the context.

Raw user request:
$ARGUMENTS

## Context Gathering (executor does this BEFORE spawning the advisor)

The advisor cannot see your conversation. You must build the full picture. Gather two types of context:

### 1. Conversation context (three layers)

1. **Task summary** (1-2 sentences): What was the goal? What did the user ask for?
2. **What was done** (bullet points, compressed): What approach was taken? Key decisions made? Files created/modified and why? Keep earlier steps compressed to essentials.
3. **Recent context** (raw, uncompressed): The last 3-5 tool calls and results — especially test outputs, error messages, and the final state of implementation. Include these **verbatim**. The advisor needs to see exactly what happened to review accurately.

### 2. Git diff context

1. Run `git status` to see untracked/staged files
2. Run `git diff --stat` to get the list of changed files and line counts
3. Run `git diff` to get the full diff (if very large, use `git diff --stat` summary only and list the top changed files)
4. If there are staged changes, also run `git diff --cached`

Structure the advisor prompt as:

```
## Task
[1-2 sentence summary of the session goal]

## What Was Done
- [compressed summary of approach and key decisions]
- [files created/modified and why]

## Recent Context (verbatim)
[last 3-5 tool calls/results, test outputs, errors]

## Changes (git)
[git status, git diff --stat, git diff output]

## Review Target
[user's $ARGUMENTS, or "Review all changes for correctness, edge cases, and silent failures"]
```

## Routing Rules

1. Build the full context (conversation + git diff) as described above.
2. Forward to the `advisor-opus:opus-advisor` subagent with the structured context, plus this prefix:

   "Review the following code changes and context. This is a review task — provide thorough, detailed analysis without the usual 100-word limit. Focus on:
   1. **Correctness** — bugs, logic errors, unhandled edge cases
   2. **Architecture** — design flaws, coupling, scalability concerns
   3. **Security** — vulnerabilities, data exposure, injection risks
   4. **Silent Failures** — swallowed errors, missing validation, unsafe defaults

   If the implementation is sound, confirm it concisely. Do not invent problems.
   Use Read/Glob/Grep to examine full file context when the diff alone is insufficient to judge correctness."

3. If `$ARGUMENTS` is provided, include it as the `## Review Target` section.
4. Return the advisor's review verbatim.

## How the Executor Should Treat the Review

Give the review serious weight. If you have primary-source evidence that contradicts a finding (test output, file content), don't silently dismiss it — surface the conflict in a follow-up advisor call to reconcile.

**Important**: Before calling this review, make your deliverable durable — write the file, save the result, commit the change. The advisor call takes time; if the session ends during it, a durable result persists and an unwritten one doesn't.
