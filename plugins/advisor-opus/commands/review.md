---
description: Have Opus review code changes or architecture. Opus sessions — explicit /review only. Sonnet/Haiku — auto-suggest when code-complete (files written, tests run, about to commit). Complements tests ("does it work") with advisor ("did we miss anything"). Skip trivial changes. Manual always works.
argument-hint: '[what to review — files, approach, architecture, etc.]'
context: fork
allowed-tools: Agent, Bash
---

Route this request to the `advisor-opus:opus-advisor` subagent with a review-specific framing.

**IMPORTANT**: The advisor agent starts fresh with NO conversation history. You (the executor) must build the context.

Raw user request:
$ARGUMENTS

## Context Gathering (executor does this BEFORE spawning the advisor)

The advisor starts with zero context — it cannot see your conversation at all. You must pass it everything it needs.

### 1. Conversation context (three steps, follow in order)

**Step 1 — Task summary** (1-2 sentences):
Write what the goal was. Example: "User asked to add rate limiting to the API endpoints."

**Step 2 — What was done** (bullet list, keep short):
List approach taken, key decisions, files created/modified and why. Compress into short bullets.

**Step 3 — Recent raw output** (copy-paste, do NOT summarize):
Copy the last 3-5 tool results exactly as they appeared — especially test outputs, error messages, and the final state. The advisor needs the actual text to review accurately. If a tool returned 50 lines, include all 50 lines.

### 2. Git diff context

1. Run `git status` to see untracked/staged files
2. Run `git diff --stat` to get the list of changed files and line counts
3. Run `git diff` to get the full diff (if very large, use `git diff --stat` summary only and list the top changed files)
4. If there are staged changes, also run `git diff --cached`

**If the working tree is clean and there are no staged changes:**
- If `$ARGUMENTS` is provided, skip the diff requirement and proceed with the review using the arguments as the review target (the user may want to review specific files, architecture, or an already-committed change).
- If `$ARGUMENTS` is empty, try the last commit as fallback: run `git diff HEAD~1 --stat` and `git diff HEAD~1`. If that produces output, use it as the diff context and note to the advisor: "These are the changes from the most recent commit (already committed)."
- Only if both `$ARGUMENTS` is empty AND `git diff HEAD~1` produces no useful output, tell the user: "No changes found to review. Try specifying a target: `/advisor-opus:review <file, topic, or commit range>`"

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
   Use Read/Glob/Grep ONLY to read specific files already named in the diff — do NOT read files unrelated to the changes, and do NOT do a full codebase scan."

3. If `$ARGUMENTS` is provided, include it as the `## Review Target` section.
4. Return the advisor's review verbatim.

## How the Executor Should Treat the Review

Give the review serious weight. If you have primary-source evidence that contradicts a finding (test output, file content), don't silently dismiss it — surface the conflict in a follow-up advisor call to reconcile.

**Important**: Before calling this review, make your deliverable durable — write the file, save the result, commit the change. The advisor call takes time; if the session ends during it, a durable result persists and an unwritten one doesn't.
