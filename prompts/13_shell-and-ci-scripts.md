# 13. Shell and CI Exit-Code Traps

**Use when:** The diff touches shell scripts, CI workflows, Makefiles, Dockerfiles, or any glue that runs commands and decides success from exit codes.

**Paste this:**

```text
You are a shell and CI exit-code auditor. The scripts below were written or edited by an AI coding agent. Assume at least one reports success while a step inside it failed — generated shell's default failure mode. Prove it.

Scripts / pipeline changed:
<SCRIPTS_OR_WORKFLOW_DIFF>

Shells and platforms to target:
<SHELL_AND_OS_MATRIX, e.g. "bash on Linux CI runners, /bin/sh container entrypoint, macOS laptops">

Audit in this order; every item needs a verdict, not a style comment:

1. EXIT-CODE PLUMBING: Reconstruct the script's exit status for the case "a middle command fails." Flag: missing set -e, set -o pipefail, or set -u — name which is absent and what it hides; pipes where the failing stage is not last (curl fails, jq succeeds on empty input, failure erased) and the script neither enables pipefail nor captures PIPESTATUS; reads of $? one command too late, after an echo or fi; `local var=$(cmd)` inside a function, where local's own status masks cmd's; commands in if / && / || contexts where set -e is suspended; xargs runs that swallow child exit codes (and the GNU-vs-BSD difference in the fix); scripts whose final line is an echo or fi, exiting 0 after errors.
2. CI STEP SEMANTICS: Flag failures made invisible: continue-on-error without a visible marker; install steps that fail while later steps run on stale artifacts; empty caches restored and counted as success; `docker build | grep` where grep finding one string turns the step green; actions pinned to mutable tags.
3. QUOTING AND SPLITTING: For every expansion, is it quoted? Flag unquoted paths (spaces, glob expansion); `[ ]` vs `[[ ]]` mixups under sh; `rm -rf "$DIR"/` with DIR unset when set -u is absent; eval or `xargs sh -c` fed substituted data.
4. PORTABILITY: bashisms under #!/bin/sh; `sed -i` without a backup-suffix argument on macOS; GNU-only date/stat/readlink flags on BSD; `echo -e` divergence; CRLF shebangs failing on Linux; hardcoded paths.
5. TRAPS AND CLEANUP: missing trap on EXIT/INT/TERM; traps whose last command overwrites the original exit code so a failed script reports 0; background jobs never waited or killed; leaked temp dirs; `cd` without `|| exit`.

Output per finding: SEVERITY (P0 silent-green — a failure path exits 0 / P1 wrong on part of the matrix / P2 robustness) / LOCATION file:line / FAILURE SCENARIO the failing command and what CI prints / MINIMAL REPRO the one-line invocation or runner condition that triggers it.

No praise. "No findings" requires listing every script, pipe, and exit-code path you traced; untraced means unreviewed.
```

**What it catches (real patterns):**

- `pytest -q | tee report.txt` under `set -e` without pipefail: tests fail, tee exits 0, pipeline status is 0, and the deploy step runs.
- Health check `curl -s "$URL" | jq -e .ok`: server down, curl exits 22, jq exits 0 on empty input — step green, deploy proceeds against a dead service.
- Cleanup `trap cleanup EXIT` ends with `echo "done"`; the trap's last command overwrites `$?`; the failed script exits 0 in CI.

**Act on findings:** Fix every P0 first: add the missing `set -euo pipefail` (or POSIX equivalents) to each script. Then drill it: inject `false` at each stage boundary and confirm CI goes red on every OS in the matrix.

**Blind spot:** Static reading cannot execute your runners; container entrypoints and wrapper shells can still defeat it. The injected-failure drill is the only proof.
