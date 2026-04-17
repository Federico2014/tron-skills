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
[TAG] Problem description.
Suggestion or corrected snippet (if applicable).
```

Tags and their meaning:

| Tag | Meaning | Blocks merge? |
|-----|---------|--------------|
| `[MUST]` | Correctness or security issue — must fix | Yes |
| `[SHOULD]` | Strongly recommended — affects quality or maintainability | Recommended |
| `[NIT]` | Minor detail — can be skipped | No |
| `[QUESTION]` | Pure question, no change required | No |
| `[DISCUSS]` | Needs discussion before deciding | Depends on conclusion |

**Comment writing rules**:
- Be direct: state what the problem is, where, and how to fix it — no lengthy preamble
- Group all minor issues of the same type (e.g., formatting) into a single comment instead of multiple separate ones
- If deep discussion is needed, move it to the PR main thread, not inline

Output rules:
- No `>` blockquote prefix — plain text for direct GitHub copy-paste
- One finding per block, blank line between blocks
- Omit files with no issues
- End with one-line verdict: `LGTM`, `LGTM with nits`, or `Changes requested (N [MUST] blockers)`

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

### PR Scope
- More than 10 non-test files changed across 2+ unrelated modules → flag for splitting (`[MUST]` if clearly unrelated, `[SHOULD]` otherwise)
- Exception: bulk code relocation, or new feature with pre-approved design

### Security
- Input not validated at boundaries (user input, external APIs, RPC params)
- Signature / crypto logic errors — wrong curve, missing verification, malleability
- `java.lang.Math` or `Maths` used directly → must use `org.tron.common.math.StrictMathWrapper`
- Sensitive data logged or exposed in error messages

### Numeric Safety
- Floating-point arithmetic in consensus code → must use `BigDecimal` or `BigInteger`
- Integer arithmetic without overflow protection → use `StrictMathWrapper.xxxExact` series
- Type casting without range check → use `Math.toIntExact` / `xxxExact` equivalents
- Fee / energy / bandwidth calculation overflow or underflow

### Business Logic
- Actuator `validate()` missing preconditions; `execute()` mutating state without prior validation
- `Any.unpack()` missing type guard
- Partial state updates leaving chain in inconsistent state
- Consensus-related change with no hard fork impact explanation

### Behavioral Compatibility
- Protocol / consensus change detectable by other nodes (potential mainnet fork)
- HTTP / gRPC / JSON-RPC response shape change
- DB schema change without backward compatibility explanation
- Config file change (`config.conf` etc.) not updated in sync
- Implicit invariants broken: ordering, idempotency, cardinality

### Lifecycle & Resource Safety
- `open` / `start` without paired `close` / `shutdown` in all paths (normal + exception + early return)
- DB iterators, streams, connections not closed in `finally`
- Resources created in constructor but released only conditionally

### Concurrency
- Non-thread-safe collections (`HashMap`, `ArrayList`) on shared mutable state
- Race between task `submit` and executor shutdown
- Missing `volatile`, improper `synchronized` scope

### Code Quality
- Debug artifacts: `System.out.println`, leftover `TODO`, temporary comments
- High-frequency path with redundant `INFO`/`WARN` logs
- Complex logic without comments
- Ambiguous or misleading variable / method / class names

### Test Quality
- Missing edge cases: null, empty, boundary, concurrent access
- Flaky patterns: `Thread.sleep`, wall-clock assertions, shared static state
- Tests that mock so deep they no longer exercise real logic
- Only happy-path coverage — no exception or boundary branches

### Code Style (framework / plugins modules only)
- Line > 100 chars, star imports, wrong indentation (2 spaces, 4 for continuation)
- Commit subject > 50 chars or wrong format: `type(scope): lowercase-verb …`

### AI-Generated Code Signals
Flag with `[SHOULD]` when multiple of the following appear together:
- Style inconsistency within the same PR — some sections feel unusually "polished"
- Functionally correct but introduces unnecessary object creation, lock contention, or I/O on hot paths
- Comments are detailed but don't accurately describe the actual implementation
- Excessive defensive code: redundant null checks, repeated boundary validation
- Happy-path tests look complete but exception and edge branches are missing

When 3+ signals are present, add a `[MUST]` asking the submitter to explain the logic section by section.
