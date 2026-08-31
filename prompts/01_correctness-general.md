# 01. Correctness General
**Use when:** Before merging any meaningful diff, especially code you did not write or cannot explain line by line. This is the master gate for behavioral correctness and regression risk.

**Paste this:**
```text
Act as the final adversarial correctness gate for this change. Assume the diff is broken in at least one consequential way. Do not praise the code, summarize it for its own sake, or suggest cosmetic improvements.

Inputs:
- Intended behavior and acceptance criteria: <INTENDED_BEHAVIOR>
- Diff or changed files: <DIFF>
- Relevant surrounding code, contracts, and repository rules: <REPO_CONTEXT>
- Available test commands and observed results: <TEST_EVIDENCE>

Trace every changed behavior from external input to observable output or side effect. Compare the implementation against the stated intent, existing callers, data invariants, API contracts, state transitions, and failure semantics. Hunt specifically for wrong conditions, inverted logic, stale assumptions, missing branches, incorrect defaults, units or type confusion, mutation/aliasing, order dependence, partial updates, compatibility regressions, and behavior that only appears correct on the happy path. Inspect unchanged code when needed to validate a changed call site or contract. Treat tests as claims, not proof: identify important paths they do not exercise and tests whose assertions would pass under the suspected bug.

For each real finding, output exactly:
SEVERITY: BLOCKER | HIGH | MEDIUM | LOW
LOCATION: <file:line or smallest identifiable symbol/hunk>
FAILURE SCENARIO: <specific preconditions, execution path, and incorrect observable result>
MINIMAL REPRO: <smallest concrete test, command, or input sequence that should expose it>

Rank findings by user or system impact. Do not report style, naming, speculative redesigns, or issues outside the diff's causal blast radius. A finding is valid only if you can state the violated requirement or invariant and a plausible execution path. If context is missing, name the exact missing artifact and explain which correctness judgment it blocks; do not invent facts.

You may conclude NO FINDINGS only after a CHECKED section listing the concrete behaviors, branches, callers, invariants, failure paths, and test gaps you examined. Then provide RESIDUAL RISK with the most important thing that could not be established from the supplied evidence. Never use passing tests alone as justification for NO FINDINGS.
```

**What it catches (real patterns):**

- A renamed status value reaches one producer but not an existing consumer → valid records silently fall into the consumer’s default branch.
- A condition changes from `<` to `<=` while pagination remains zero-based → the final item is duplicated only on full pages.
- A function mutates an object previously treated as immutable → cached callers observe cross-request state contamination despite isolated unit tests passing.

**Act on findings:** Fix BLOCKER and HIGH findings before merge, starting with violated contracts and corrupted state. Add the minimal repro as a regression test. Reject the whole diff when its core approach contradicts the acceptance criteria or multiple findings share a faulty design assumption.

**Blind spot:** This gate cannot prove requirements that are absent or validate runtime dependencies it cannot inspect. Domain, production-load, and environment-specific failures still require representative evidence.
