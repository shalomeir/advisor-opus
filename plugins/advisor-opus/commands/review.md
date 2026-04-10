---
description: Have Claude Opus review current code changes, architecture decisions, or implementation approach. Focuses on correctness, edge cases, and silent failures.
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

   "Review the following with a critical eye. Focus on:
   1. **Correctness** — bugs, logic errors, unhandled edge cases
   2. **Architecture** — design flaws, coupling, scalability concerns
   3. **Security** — vulnerabilities, data exposure, injection risks
   4. **Silent Failures** — swallowed errors, missing validation, unsafe defaults

   If the implementation is sound, confirm it concisely. Do not invent problems.

   Review target: [user's request]"

3. **Important**: Tell the advisor that this is a review task — it should provide thorough, detailed analysis without the usual 150-word limit. Structured sections with file-specific findings are preferred.
4. Return the advisor's review verbatim.
