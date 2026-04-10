---
description: Have Claude Opus review code changes, architecture decisions, or implementation approach. Auto-suggest for complex multi-step tasks (3+ steps, multi-file changes) when work is code-complete — all files written, tests run, about to commit. Complements verification (tests check "does it work", advisor checks "did we miss anything"). SKIP for trivial changes. Manual /review always works.
argument-hint: '[what to review — files, approach, architecture, etc.]'
context: fork
allowed-tools: Agent
---

Route this request to the `advisor-opus:opus-advisor` subagent with a review-specific framing.

Raw user request:
$ARGUMENTS

## Routing Rules

1. If `$ARGUMENTS` is empty, forward with: "Review the files in the working directory. Use Glob and Read to explore the codebase structure, then review key files for correctness, edge cases, silent failures, and security issues. Be specific — name files and line numbers."
2. If `$ARGUMENTS` is provided, forward to the `advisor-opus:opus-advisor` subagent with this prefix:

   "Review the following with a critical eye. This is a review task — provide thorough, detailed analysis without the usual 100-word limit. Focus on:
   1. **Correctness** — bugs, logic errors, unhandled edge cases
   2. **Architecture** — design flaws, coupling, scalability concerns
   3. **Security** — vulnerabilities, data exposure, injection risks
   4. **Silent Failures** — swallowed errors, missing validation, unsafe defaults

   If the implementation is sound, confirm it concisely. Do not invent problems.

   Review target: [user's request]"

3. Return the advisor's review verbatim.

## How the Executor Should Treat the Review

Give the review serious weight. If you have primary-source evidence that contradicts a finding (test output, file content), don't silently dismiss it — surface the conflict in a follow-up advisor call to reconcile.

**Important**: Before calling this review, make your deliverable durable — write the file, save the result, commit the change. The advisor call takes time; if the session ends during it, a durable result persists and an unwritten one doesn't.
