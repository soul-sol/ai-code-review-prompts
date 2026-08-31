# 12. Test Quality That Can Fail

**Use when:** The diff adds or edits tests — especially agent-written ones that passed on the first run. Tests written by the agent that wrote the bug tend to encode the bug.

**Paste this:**

```text
You are auditing tests, not the feature. The tests below were written by an AI coding agent to verify its own change. Assume at least one cannot fail even with the feature broken, and at least one mock lies. Find both.

Test files:
<TEST_FILES>

Code under test and spec:
<CODE_UNDER_TEST>
<TICKET_OR_SPEC>

For each test file, answer adversarially with line references:

1. CAN IT FAIL? For each assertion, name the wrong behavior that would make it fail. If you cannot name one, it is coverage theater. Flag: assertions on a mock's return value (proving the mock, not the code); expect(x).toBeDefined() or toBeTruthy() on anything; snapshots updated in the commit they were meant to guard; assertions inside loops that never iterate; setup that throws before the assert while the runner still counts the test green (check skipped-vs-errored semantics).
2. DOES IT INTERROGATE THE CODE? Flag tests that mock the entire unit under test, assert a call happened without checking arguments or effects, or pin implementation details (call counts, private names) that can hold while output is wrong.
3. DO THE MOCKS LIE? Compare each mock's shape against the real dependency in <CODE_UNDER_TEST>. Flag: mocks that always succeed where the real thing can fail; mocks returning data where the real API returns null or empty; wrong field names, casing, or units; time and randomness mocked so ordering and retry bugs become untestable.
4. IS THE CONTRACT RIGHT? The agent may have tested its own bug. For each behavioral assertion, check it against <TICKET_OR_SPEC>. A test that faithfully pins wrong behavior is the worst finding here — mark it P0.
5. UNTESTED FAILURE MODES: List error paths, boundary values, cancellation, concurrent calls, empty input, and retry behavior with zero coverage. Name the three most dangerous untested paths.
6. HYGIENE: shared fixtures and order dependence; network/timezone/locale dependence; async tests missing await (a rejected promise nobody awaits is a silent pass); test logic that reimplements the algorithm so both are wrong together.

Output per finding: SEVERITY (P0 validates wrong behavior or can never fail / P1 lying mock or critical missing path / P2 hygiene) / LOCATION file:line / FAILURE SCENARIO the production bug this test waves through / MINIMAL REPRO the code change that breaks production while this suite stays green.

No praise. "No findings" requires listing every test file and assertion class examined — plus the three most dangerous untested paths regardless.
```

**What it catches (real patterns):**

- Every mock returns the happy shape; the suite is green while production 500s on a field the real API omits — the mock lied about nullability.
- An async test is missing `await`: the assertion never runs, the promise rejects after the runner moved on, CI reports pass.
- The agent's test asserts the off-by-one it introduced — the sentinel row inside a page of 10 — so the bug is pinned as correct behavior.

**Act on findings:** Delete or rewrite P0 tests before merge; a suite that cannot fail is negative value. Turn item 5's top three gaps into failing-first tests. If mocks outnumber real assertions, reject the diff's verification claim outright.

**Blind spot:** It reads tests; it cannot run them. A structurally sound test can still miss bugs this prompt cannot imagine — for high stakes, break the code on purpose once and confirm the suite notices.
