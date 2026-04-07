---
name: tron-issue-comment
description: >
  Write copy-paste-ready GitHub issue comments for tronprotocol/java-tron.
  Covers: bug analysis, feature request feedback, needs-more-info, and close/resolve replies.
  Use when the user provides a GitHub issue URL, a comment URL, or asks to reply to an issue on java-tron.
  Trigger phrases: "write an issue comment", "reply to this issue", "帮我回复这个issue", "/tron-issue-comment".
  Also triggers when the user pastes a GitHub URL matching: github.com/tronprotocol/java-tron/issues/*
---

# tron-issue-comment

## Workflow

### Step 1 — Fetch the issue content

**If the user provides a URL**, parse it and fetch accordingly:

```
# Issue URL: https://github.com/tronprotocol/java-tron/issues/1234
gh issue view 1234 --repo tronprotocol/java-tron --comments

# Comment URL: https://github.com/tronprotocol/java-tron/issues/1234#issuecomment-987654321
# → fetch the issue + all comments, then locate the specific comment by ID
gh issue view 1234 --repo tronprotocol/java-tron --comments
```

If no URL is provided, ask the user to paste the issue title, body, or thread.

### Step 2 — Identify context

- Read the full issue thread (title + body + existing comments).
- If replying to a specific comment, note what that comment said.
- Auto-detect the comment type below; ask only if genuinely ambiguous.

### Step 3 — Draft and output

Use the matching template. Output only the comment body — no preamble, no explanation.

---

## Comment Types & Templates

### Bug Analysis

Use when: a bug is reported and you have enough info to analyze it.

```
Thanks for the report.

**Root cause**
<one or two sentences explaining what's wrong and where>

**Affected component**
`module/path/ClassName.java` — <brief description of the problematic code path>

**Fix direction**
<concise description of the correct fix; include a code snippet if helpful>

This will be tracked for the next patch. Feel free to submit a PR — happy to review.
```

---

### Feature Request Feedback

Use when: evaluating whether a feature should be accepted, deferred, or redirected.

```
Thanks for the proposal.

**Assessment**
<feasibility and impact in 2–3 sentences — be direct about whether this is viable>

**Concerns / trade-offs**
- <concern 1>
- <concern 2>

**Recommendation**
<Accept / Defer / Needs TIP / Won't implement> — <one-sentence rationale>

<If TIP needed:> This change affects the protocol layer and should go through a TIP first. See https://github.com/tronprotocol/tips for the process.
```

---

### Needs More Info

Use when: the issue lacks reproduction steps, version info, logs, or other detail needed to triage.

```
Thanks for filing this.

To investigate further, could you provide:

- **java-tron version**: (`git log -1 --oneline` or release tag)
- **Network**: Mainnet / Nile testnet / private net
- **Reproduction steps**: minimal sequence to trigger the issue
- **Logs**: relevant output from `logs/tron.log` around the time of the error (stack trace if available)
- **Node config**: any non-default settings in `config.conf` related to this area

Once we have the above we can take a closer look.
```

---

### Close / Resolve

Use when: closing a bug as fixed, won't-fix, duplicate, or out of scope.

```
<Choose one opener:>
Fixed in <commit / PR link> — this will be included in the next release.
Closing as duplicate of #<number>.
Closing as won't-fix: <one sentence reason>.
Closing as out of scope: <one sentence reason>.

<Optional follow-up sentence if action is needed from the reporter.>

Feel free to reopen if you run into this again with the details above.
```

---

## Output Rules

- Plain text only — no `>` blockquote prefix, no HTML
- English (match the language of the issue thread if it's Chinese)
- Remove unused placeholder lines — output only what applies
- Do not wrap the comment in a code block unless showing a code example within it
- Output only the comment body — no "Here is the comment:" preamble
