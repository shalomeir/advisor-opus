---
description: Have Claude Opus review code changes, architecture decisions, or implementation approach. Auto-suggest for complex multi-step tasks (3+ steps, multi-file changes) when work is code-complete — all files written, tests run, about to commit or declare done. Complements verification (tests check "does it work", advisor checks "did we miss anything"). SKIP for trivial changes. Manual /review always works.
argument-hint: '[what to review — files, approach, architecture, etc.]'
context: fork
allowed-tools: Agent, Bash
---

Route this request to the `advisor-opus:opus-advisor` subagent with a review-specific framing.

Raw user request:
$ARGUMENTS

## Pre-Processing (executor does this BEFORE spawning the advisor)

Before forwarding to the advisor, gather diff context so the advisor knows exactly what changed:

1. Run `git diff --stat` to get the list of changed files and line counts
2. Run `git diff` to get the full diff (if very large, use `git diff --stat` summary only and list the top changed files)
3. Run `git status` to see untracked/staged files
4. If there are staged changes, also run `git diff --cached`

Include ALL of this output in the advisor prompt under a `## Changes` section.

## Routing Rules

1. If `$ARGUMENTS` is empty, forward to the `advisor-opus:opus-advisor` subagent with this prompt:

   "Review the following code changes. This is a review task — provide thorough, detailed analysis without the usual 100-word limit. Focus on:
   1. **Correctness** — bugs, logic errors, unhandled edge cases
   2. **Architecture** — design flaws, coupling, scalability concerns
   3. **Security** — vulnerabilities, data exposure, injection risks
   4. **Silent Failures** — swallowed errors, missing validation, unsafe defaults

   If the implementation is sound, confirm it concisely. Do not invent problems.
   Use Read/Glob/Grep to examine full file context when the diff alone is insufficient to judge correctness.

   ## Changes
   [paste git status, git diff --stat, and git diff output here]"

2. If `$ARGUMENTS` is provided, forward to the `advisor-opus:opus-advisor` subagent with the same review prefix, adding the user's request as the review target. Still include the diff context in `## Changes`.

3. Return the advisor's review verbatim.

## How the Executor Should Treat the Review

Give the review serious weight. If you have primary-source evidence that contradicts a finding (test output, file content), don't silently dismiss it — surface the conflict in a follow-up advisor call to reconcile.

**Important**: Before calling this review, make your deliverable durable — write the file, save the result, commit the change. The advisor call takes time; if the session ends during it, a durable result persists and an unwritten one doesn't.
