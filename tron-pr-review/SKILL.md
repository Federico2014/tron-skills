---
name: tron-pr-review
description: >
  PR code review for java-tron (tronprotocol/java-tron). Two modes: (1) full code review
  producing copy-paste-ready inline comments grouped by file with verified line numbers and
  severity tags; (2) reply to a specific PR comment. Use when asked to review a PR, review
  a diff, or reply to someone's comment on a java-tron PR.
  Trigger phrases: "review this PR", "帮我review", "/tron-pr-review".
  Also triggers when the user pastes a URL matching: github.com/tronprotocol/java-tron/pull/*
---

# tron-pr-review

## Mode Detection

Determine mode from what the user provides:

| Input | Mode |
|---|---|
| PR URL / number / branch | **Full review** |
| PR comment URL (`#issuecomment-*` or `#discussion_r*`) | **Reply to comment** |

---

## Mode 1 — Full Review

### Step 1: Fetch the diff

```bash
# From PR URL or number
gh pr diff <number> --repo tronprotocol/java-tron

# From local branch
git diff develop..HEAD
```

### Step 2: Verify line numbers

Read the actual file content with the Read tool for every finding. Never trust diff offsets blindly.

### Step 3: Output inline comments

Group by file. For each finding:

```
**path/to/File.java:LINE**
[Severity] Problem description.
Suggestion or corrected snippet (if applicable).
```

Severity: `[Critical]` `[High]` `[Medium]` `[Low]` `[Nit]`

- No `>` blockquote prefix — plain text for direct GitHub copy-paste
- One finding per block, blank line between blocks
- Omit files with no issues
- End with one-line verdict: `LGTM`, `LGTM with nits`, or `Changes requested (N blockers)`

---

## Mode 2 — Reply to Comment

### Step 1: Fetch the comment and context

```bash
# Fetch the full PR thread (comments + review comments)
gh pr view <number> --repo tronprotocol/java-tron --comments

# For review (inline) comments, also fetch:
gh api repos/tronprotocol/java-tron/pulls/<number>/comments
```

Locate the specific comment by ID from the URL fragment (`#issuecomment-<id>` or `#discussion_r<id>`).

### Step 2: Draft a reply

- Address the commenter's point directly — agree, disagree, or ask for clarification
- If disagreeing: cite the code path, spec, or rationale
- If agreeing: state what the fix will be or reference the relevant commit/PR
- Keep it concise; no need to restate what the commenter said

### Step 3: Output

Plain text only, ready to paste. No preamble, no "Here is the reply:" wrapper.

---

## Review Checklist (Mode 1)

Work through each category. Skip silently if nothing found.

### Security
- Input not validated at boundaries (user input, external APIs, RPC params)
- Signature / crypto logic errors — wrong curve, missing verification, malleability
- `java.lang.Math` used directly → must use `org.tron.common.math.StrictMathWrapper`
- Sensitive data logged or exposed in error messages

### Business Logic
- Actuator `validate()` missing preconditions; `execute()` mutating state without prior validation
- Fee / energy / bandwidth calculation overflow or underflow
- `Any.unpack()` missing type guard
- Partial state updates leaving chain in inconsistent state

### Behavioral Compatibility
- Protocol / consensus change detectable by other nodes (potential mainnet fork)
- HTTP / gRPC / JSON-RPC response shape change
- Implicit invariants broken: ordering, idempotency, cardinality

### Lifecycle & Resource Safety
- `open` / `start` without paired `close` / `shutdown` in all paths (normal + exception + early return)
- DB iterators, streams, connections not closed in `finally`
- Resources created in constructor but released only conditionally

### Concurrency
- Non-thread-safe collections (`HashMap`, `ArrayList`) on shared mutable state
- Race between task `submit` and executor shutdown
- Missing `volatile`, improper `synchronized` scope

### Test Quality
- Missing edge cases: null, empty, boundary, concurrent access
- Flaky patterns: `Thread.sleep`, wall-clock assertions, shared static state
- Tests that mock so deep they no longer exercise real logic

### Code Style (framework / plugins modules only)
- Line > 100 chars, star imports, wrong indentation (2 spaces, 4 for continuation)
- Commit subject > 50 chars or wrong format: `type(scope): lowercase-verb …`
