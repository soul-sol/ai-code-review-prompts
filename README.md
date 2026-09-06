# AI Code Review Prompts (free set)

**Adversarial review prompts for AI-written code.** Your agent wrote the diff in four minutes — it compiles, tests are green, and the summary sounds confident. None of that tells you what it broke.

Each prompt turns any strong code model (Claude, GPT/codex, etc.) into a hostile reviewer with one job: **assume the diff is broken, and prove it.** Findings come back in a fixed format — `SEVERITY / LOCATION / FAILURE SCENARIO / MINIMAL REPRO` — and praise is forbidden. "No findings" is only accepted together with a list of what was actually checked.


> These came out of running Claude Code and Codex workers in parallel every day.
> The incidents behind them — what the agent claimed, what actually happened, and the
> gate that catches it next time — are at
> [status.lifestep.io/incidents](https://status.lifestep.io/incidents/).
> The full set is [The Adversarial Review Prompt Pack](https://lifestep1.gumroad.com/l/adversarial-review-prompt-pack) ($7);
> what is here stays free and MIT either way.

## The 5 free prompts

| # | Prompt | Hunts for |
|---|---|---|
| 01 | [Correctness (master gate)](prompts/01_correctness-general.md) | the flagship adversarial review — run this before any merge |
| 05 | [Security: injection](prompts/05_security_injection.md) | SQLi, XSS, command injection, path traversal |
| 12 | [Test quality](prompts/12_test-quality.md) | tests that cannot fail, lying mocks, coverage theater |
| 13 | [Shell & CI exit-code traps](prompts/13_shell-and-ci-scripts.md) | pipefail/PIPESTATUS, silent-green CI, quoting, portability |
| 22 | [Agent-output verification](prompts/22_agent-output-verification.md) | a worker claims DONE — audit the diff against its brief |

## How to run (non-interactive)

```bash
git diff main...HEAD > /tmp/diff.txt
claude -p "$(sed -n '/```text/,/```/p' prompts/01_correctness-general.md | sed '1d;$d')

<DIFF>
$(cat /tmp/diff.txt)"
```

Works the same with `codex exec` or any CLI that accepts a prompt. Model-agnostic.

## The full pack (25 prompts)

The complete set adds: concurrency/races, error handling & partial failure, authz/IDOR, secrets & logging, performance hot paths, **DB migration safety**, API compatibility, frontend regressions, dependency upgrades, refactoring equivalence, scope creep, **LLM-app prompt injection & token cost**, data pipelines, infra/config, git hygiene, rollback/deploy safety, docs drift, and a **two-model second-opinion protocol** — plus a 55-page PDF with a severity rubric, triage order, and a "reject the whole diff" checklist (EN + Korean guide).

**→ [$7 on Gumroad](https://lifestep1.gumroad.com/l/adversarial-review-prompt-pack)**

Related: [agent-watch](https://github.com/soul-sol/agent-watch) (stall detection for background agents) · [Solo, Like a Team — Claude Code Multi-Agent Orchestration in Practice](https://lifestep1.gumroad.com/l/solo-like-a-team-claude-code-orchestration) (the book these come from) · [free orchestration templates](https://github.com/soul-sol/claude-code-orchestration-ko)

## License

The 5 prompt files in this repository are MIT licensed — use them anywhere, including commercially. The paid pack has its own terms.

<!-- xlink:start -->
## Related free tools

- [XLSX Inspector](https://xlsx.lifestep.io) — check workbooks for macros, external links and hidden sheets
- [DNS and SPF Check](https://dnscheck.lifestep.io) — records, SPF, DMARC and TLS expiry
- [Email Validator](https://emailcheck.lifestep.io) — syntax, MX, disposable and role addresses
- [QR Code Generator](https://qrcode.lifestep.io) — free PNG and SVG API, no signup
- [agent-watch](https://github.com/soul-sol/agent-watch)
- [claude-md-patterns](https://github.com/soul-sol/claude-md-patterns)
- [claude-code-orchestration-ko](https://github.com/soul-sol/claude-code-orchestration-ko)
- [xlsx-inspector-api](https://github.com/soul-sol/xlsx-inspector-api)
- [domain-info-api](https://github.com/soul-sol/domain-info-api)
- [email-validator-api](https://github.com/soul-sol/email-validator-api)
- [qr-code-api](https://github.com/soul-sol/qr-code-api)

The paid guide collection is available at [lifestep1.gumroad.com](https://lifestep1.gumroad.com).
<!-- xlink:end -->
