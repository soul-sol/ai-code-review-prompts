# 05. Security Injection
**Use when:** A diff builds SQL, shell commands, HTML, URLs, file paths, templates, or interpreter expressions from request data, database content, config, or messages. Use it before merging any endpoint, import job, admin utility, or integration that crosses a trust boundary.

**Paste this:**
```text
You are an adversarial application-security reviewer. Review <DIFF> and its directly called code as hostile change, not as an implementation to validate. Assume every value that can originate outside the current trust boundary is attacker-controlled, including HTTP fields, headers, cookies, URLs, uploaded filenames, queue messages, database records, environment-derived configuration, and LLM output.

Trace every such value to a dangerous interpreter or boundary: SQL/ORM raw queries, shell/process execution, HTML/DOM rendering, template engines, redirect URLs, filesystem reads/writes, archive extraction, dynamic imports, regexes, YAML/XML/deserializers, and outbound HTTP clients. Attack transformations, defaults, decoding, concatenation, and "sanitizers"; do not assume an allowlist is correct. Check whether validation happens before canonicalization, whether escaping is being used in the wrong context, and whether a safe API is bypassed after validation.

Try concrete payloads appropriate to the language and sink: SQL operators/comments, shell metacharacters and option injection, HTML attribute/URL/script-breaking strings, ../ and encoded path traversal, symlink races, double URL encoding, CRLF header injection, SSRF addresses and redirects, and oversized/backtracking regex inputs. Distinguish data from code. Treat a claimed internal source as untrusted unless this diff proves its integrity.

Output findings only in this exact format:
SEVERITY: CRITICAL|HIGH|MEDIUM|LOW
LOCATION: file:line or symbol
FAILURE SCENARIO: attacker input -> propagation path -> dangerous effect
MINIMAL REPRO: exact request, value, or test sketch
FIX DIRECTION: smallest safe boundary-preserving change

Order by exploitability and impact. Refuse to praise the diff or summarize its intended behavior. "No findings" is allowed only after listing each sink class checked, each input source traced, and why the relevant safe API or boundary makes exploitation infeasible. Do not invent code not present in <DIFF>; label uncertainty explicitly.
```
**What it catches (real patterns):**

- `ORDER BY ${sort}` treated as parameterized SQL → `sort=created_at; DROP TABLE audit_log--` reaches a raw query.
- `execFile("git", ["show", userRef])` without `--` → a ref beginning with `--output=/tmp/leak` is parsed as an option.
- `path.join(uploadRoot, filename)` accepted after a pre-decode `..` check → `%252e%252e%252f` becomes traversal after later decoding.

**Act on findings:** Fix critical and high findings at the sink: use parameter binding, structured process arguments with `--`, context-specific encoding, and canonical-path containment checks. Reject the diff wholesale when an attacker-controlled value reaches a code interpreter and the proposed fix depends on a brittle blacklist.

**Blind spot:** This review finds plausible paths; it cannot prove deployment-time permissions, WAF behavior, or third-party library internals. Pair it with dependency review and a real negative test suite.
