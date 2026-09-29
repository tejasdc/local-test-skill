# Test cadence (universal)

Moved verbatim from the global instruction file on 2026-09-29; the global file keeps a two-line summary.


Choose verification by the behavior and regression surface of the change. Full suites are integration checks, not routine iteration loops; their duration alone does not decide whether they are needed.

**The rule for every agent and every project:**
- During implementation and repairs, run focused tests for the changed behavior and observed failures. Fix those failures before starting a full suite.
- Use a full suite after the integrated change when shared behavior, cross-feature interactions or a required release gate warrant it. Bounded changes with an isolated regression surface may be verified with targeted tests alone.
- If a full suite finds a regression, reproduce and fix it with targeted tests, then rerun the full suite when needed to establish integration confidence. Repeat as necessary; there is no numerical cap and no extra permission requirement merely because a suite has run before.
- Do not repeat passing checks without a source change, new failure or unresolved concern that makes the result relevant again. Explain the selected verification scope and its limits.

When a single prompt specifies multiple features, implement ALL of them first, THEN verify end-to-end. Do not implement-and-verify-and-implement-and-verify in series — that multiplies the wait.

The user explicitly does not want "polish rounds" or "follow-up rounds" — visual polish, design-system application, and feature work are ALL deliverables of the round that introduces them. If your prompt names a design system, you MUST apply it visually; don't defer.
