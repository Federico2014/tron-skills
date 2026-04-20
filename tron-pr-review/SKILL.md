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

### Step 2: Check PR size and scope

Before reviewing content, verify the PR meets the Single Module Principle:

- **File count**: non-test file changes must be ≤ 10; if exceeded, flag for splitting in the PR top-level comment
- **Module count**: if the diff spans more than 2 independent modules (net / consensus / db / API / event) with many files, recommend splitting
- **Exceptions**: wholesale code relocation, or new features that passed a prior design review — the PR description must explain the relationship

### Step 3: Verify line numbers

Read the actual file content with the Read tool for every finding. Never trust diff offsets blindly.

### Step 4: Output inline comments

Group by file. For each finding:

```
**path/to/File.java:LINE**
[TAG] Problem description.
Suggestion or corrected snippet (if applicable).
```

**Comment prefix tags**:

| Tag | Meaning | Blocks merge |
|-----|---------|-------------|
| `[MUST]` | Must fix — correctness or security issue | Yes |
| `[SHOULD]` | Strongly recommended — quality or maintainability | Recommended yes |
| `[NIT]` | Minor polish — optional | No |
| `[QUESTION]` | Pure question — no change required | No |
| `[DISCUSS]` | Needs discussion before deciding | Depends on outcome |

**Comment quality rules**:
- State the problem, location, and suggested fix directly — no lengthy preamble
- Group all minor style/format issues into **one** comment; do not split them across multiple comments
- Move deep discussions to the PR top-level thread, not inline comments

Output rules:
- No `>` blockquote prefix — plain text for direct GitHub copy-paste
- One finding per block, blank line between blocks
- Omit files with no issues
- End with a one-line verdict: `LGTM`, `LGTM with nits`, or `Changes requested (N [MUST] blockers)`

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
- `java.lang.Math` or the legacy `Maths` helper used directly → must use `org.tron.common.math.StrictMathWrapper`
- Sensitive data logged or exposed in error messages

### Numeric Safety
- Floating-point arithmetic in consensus code → must use `BigDecimal`; integer arithmetic must use `BigInteger` or `StrictMathWrapper.xxxExact`
- `long` / `int` arithmetic with overflow risk → use `xxxExact` methods or explicit boundary checks
- Unchecked narrowing casts → use `Math.toIntExact()` or equivalent `xxxExact` method

### Business Logic
- Actuator `validate()` missing preconditions; `execute()` mutating state without prior validation
- Fee / energy / bandwidth calculation overflow or underflow
- `Any.unpack()` missing type guard
- Partial state updates leaving chain in inconsistent state

### Behavioral Compatibility
- Protocol / consensus change detectable by other nodes (potential mainnet fork) → PR must describe hard-fork impact and the node version range that needs upgrading
- HTTP / gRPC / JSON-RPC response shape change
- Implicit invariants broken: ordering, idempotency, cardinality
- Database schema change without a compatibility statement
- Configuration change without syncing `config.conf` and related config files

### Lifecycle & Resource Safety
- `open` / `start` without paired `close` / `shutdown` in all paths (normal + exception + early return)
- DB iterators, streams, connections not closed in `finally`
- Resources created in constructor but released only conditionally

### Concurrency
- Non-thread-safe collections (`HashMap`, `ArrayList`) on shared mutable state
- Race between task `submit` and executor shutdown
- Missing `volatile`, improper `synchronized` scope

### Logging
- `INFO` / `WARN` level logs on hot paths → downgrade to `DEBUG` or remove

### Test Quality
- Missing edge cases: null, empty, boundary, concurrent access
- Flaky patterns: `Thread.sleep`, wall-clock assertions, shared static state
- Tests that mock so deep they no longer exercise real logic

### Code Style (framework / plugins modules only)
- Line > 100 chars, star imports, wrong indentation (2 spaces, 4 for continuation)
- `System.out.println`, leftover debug comments, unresolved TODOs
- Ambiguous, near-duplicate, or unreadable variable / method / class names
- Complex logic without explanatory comments

### AI-Generated Code Signals

When multiple signals below appear together, require the submitter to explain the code section by section in the PR thread, and tag the specific line with `[MUST] Please explain the implementation logic here` before deciding to approve:

| Signal | Description |
|--------|-------------|
| Style inconsistency | Code style shifts noticeably within the same PR; some sections look unusually "perfect" |
| Correct but slow | Functionality tests pass, but hot paths introduce unnecessary object allocation, lock contention, or I/O |
| Comment/code mismatch | Comments are detailed but describe logic that differs from the actual implementation |
| Defensive code pileup | Excessive null checks and repeated boundary validation that obscure understanding of real logic |
| Evasive answers | Submitter gives vague or off-topic replies when asked to explain a specific code section |
| Superficial test coverage | Many test cases, but all happy-path; boundary and error-branch coverage is thin |

---

## Review SLA & Priority

**Response time requirements**:
- First review: within **3 business days** of being assigned
- Reply to comment: within **2 business days** of the submitter's response
- Re-review: within **2 business days** after submitter updates the code

**Claim template** (post in the PR if you cannot start immediately):
```
👀 taking a look, expect to complete the first round by [date] EOD.
```

**Priority order when multiple PRs are pending**:
1. PRs labeled `hotfix` → handle immediately
2. PRs approaching their SLA deadline → prioritize
3. Small-diff, clearly scoped PRs → clear first
4. Large cross-module diffs → schedule a dedicated time block

---

## Merge Policy (gating checklist for reviewer)

A PR may be merged only when:
- **Standard PR**: ≥ 2 reviewer Approvals, at least 2 of whom are core maintainers of the affected module
- **Consensus / storage / network core path**: ≥ 3 reviewer Approvals
- All CI checks pass (build, unit tests, Checkstyle)
- All `[MUST]` comments are Resolved
- Submitter must not Approve their own PR via an alt account

**Merge strategy**:

| Strategy | Use case | Branch direction |
|----------|---------|-----------------|
| Squash and Merge | Feature development PR | feature → develop / release_xxx / master |
| Rebase and Merge | Branch sync | release_xxx → master, master → develop |
| Create a merge commit | Rarely used | — |

**Hotfix fast-track**: 1 core maintainer Approval is sufficient to merge; complete tests and documentation must be added within 7 days.
