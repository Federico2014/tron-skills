---
name: okr-advisor
description: >
  Analyze Federico's current quarter OKR from a local xlsx file, cross-reference with
  actual work output from GitHub, Slack, and Confluence, then recommend prioritized tasks.
  Parses the O→KR→P structure with weights, calculates completion status and time pressure,
  and outputs a ranked task list in the terminal.
  Trigger phrases: "what should I work on", "OKR status", "recommend tasks", "/okr-advisor".
---

# okr-advisor

Analyze incomplete OKR tasks for the current quarter. Cross-reference with GitHub, Slack, and Confluence activity to determine real progress, then recommend what to work on next.

## Usage

```bash
/okr-advisor
```

No arguments required. Automatically detects the current quarter and runs the full analysis.

## User Identity

| Field | Value |
|-------|-------|
| Name | Federico |
| GitHub | Federico2014 (personal), tronprotocol (org) |
| Slack Channel | `daily-ai-assistant` (ID: `C0ARYHYLZ3Q`) |
| Slack Bot Token | `$SLACK_BOT_TOKEN` (from env or `~/.claude/mcp.json` → slack → env) |
| Confluence Space | javatronx (`troneco.atlassian.net`) |

---

## Prerequisites

### OKR File

Local xlsx file at a fixed path:

```
/Users/tron/federico/okr/2026-Federico-OKR.xlsx
```

If the file does not exist, inform the user and stop.

### Python dependency

```bash
pip3 install openpyxl -q
```

Install automatically if missing.

### Slack Bot Token

The token is read at runtime from `~/.claude/mcp.json` under `mcpServers.slack.env.SLACK_BOT_TOKEN`. Extract it with:

```bash
SLACK_BOT_TOKEN=$(python3 -c "import json; print(json.load(open('$HOME/.claude/mcp.json'))['mcpServers']['slack']['env']['SLACK_BOT_TOKEN'])")
```

### GitHub CLI

`gh` must be authenticated with access to `tronprotocol/java-tron`.

### Atlassian MCP

Required for Confluence access. Should already be configured.

---

## Workflow

### Step 1 — Parse OKR File

Use python3 + openpyxl to parse the xlsx file.

**Select the current quarter sheet** based on today's date:
- Q1 = Jan–Mar → sheet name containing "Q1" and current year
- Q2 = Apr–Jun → sheet name containing "Q2" and current year
- Q3 = Jul–Sep → sheet name containing "Q3" and current year
- Q4 = Oct–Dec → sheet name containing "Q4" and current year

**Parse the O→KR→P three-level structure**:

| Column | Content |
|--------|---------|
| A | Objective (O) — only present on the first row of each O |
| B | O weight (e.g., 0.6) |
| C | Key Result (KR) — only present on the first row of each KR |
| D | KR weight (e.g., 0.4) |
| F | Action item (P) — one per row |
| K | Completion status: "是" = done, "否" = not done, empty = unknown |

**Build the OKR tree**:

```
O1 (weight 0.6) — description from column A
├── KR1 (weight 0.4) → composite_weight = 0.6 × 0.4 = 0.24
│   ├── P1: description [done/not done/unknown]
│   ├── P2: description [done/not done/unknown]
│   └── P3: description [done/not done/unknown]
├── KR2 (weight 0.3) → composite_weight = 0.6 × 0.3 = 0.18
│   └── ...
└── KR3 (weight 0.3) → composite_weight = 0.6 × 0.3 = 0.18
    └── ...
```

**Parsing rules**:
- Column A has a value → new Objective starts; read weight from column B
- Column C has a value → new Key Result starts; read weight from column D
- Column F has a value → new Action item (P) under the current KR; **record the row number** for write-back
- Column K: "是" → completed; "否" → not completed; empty → determine from external data
- Skip rows 1-2 (header rows)
- Weights like "补充项" (supplementary) → treat as 0.0
- Each P must store its `row_number` so Step 5 can write completion status back to the xlsx

### Step 2 — Collect Work Output (parallel)

Run all data collection in parallel.

#### 2a. GitHub Activity (gh CLI)

Compute `quarter_start` (e.g., `2026-04-01` for Q2).

```bash
# PRs authored (personal repos)
gh api "search/issues?q=author:Federico2014+type:pr+created:>={quarter_start}&per_page=50" \
  --jq '.items[] | {title: .title, number: .number, state: .state, url: .html_url, repo: (.repository_url | split("/") | .[-1])}'

# PRs authored (tronprotocol org)
gh api "search/issues?q=author:Federico2014+org:tronprotocol+type:pr+updated:>={quarter_start}&per_page=50" \
  --jq '.items[] | {title: .title, number: .number, state: .state, url: .html_url, repo: (.repository_url | split("/") | .[-1])}'

# PRs reviewed (tronprotocol org)
gh api "search/issues?q=reviewed-by:Federico2014+org:tronprotocol+type:pr+updated:>={quarter_start}&per_page=50" \
  --jq '.items[] | {title: .title, number: .number, url: .html_url, repo: (.repository_url | split("/") | .[-1])}'

# Issues involving Federico2014 (tronprotocol org)
gh api "search/issues?q=involves:Federico2014+org:tronprotocol+type:issue+updated:>={quarter_start}&per_page=50" \
  --jq '.items[] | {title: .title, number: .number, url: .html_url, repo: (.repository_url | split("/") | .[-1])}'

# Commits (personal repos)
gh api "search/commits?q=author:Federico2014+committer-date:>={quarter_start}&sort=committer-date&per_page=50" \
  --jq '.items[] | {repo: .repository.full_name, message: .commit.message, date: .commit.committer.date, url: .html_url}'
```

#### 2b. Slack: daily-ai-assistant (curl + Bot Token)

Fetch recent 2 weeks of messages:

```bash
two_weeks_ago=$(date -v-14d +%s 2>/dev/null || date -d '14 days ago' +%s)
curl -s -H "Authorization: Bearer ${SLACK_BOT_TOKEN}" \
  "https://slack.com/api/conversations.history?channel=C0ARYHYLZ3Q&limit=100&oldest=${two_weeks_ago}"
```

Extract message text from the JSON response. Filter for messages from Federico's user ID.

#### 2c. Confluence: Recent Weekly Reports (Atlassian MCP)

```
Tool: searchAtlassian
Params: query = "Federico 2026 weekly report"
```

Or use CQL for precision:

```
Tool: searchConfluenceUsingCql
Params:
  cloudId = "troneco.atlassian.net"
  cql = "space = 'javatronx' AND title ~ 'Federico' AND type = page ORDER BY lastModified DESC"
  limit = 5
```

Read the most recent 2 weekly reports to extract work items:

```
Tool: getConfluencePage
Params:
  cloudId = "troneco.atlassian.net"
  pageId = <page ID from search>
  contentFormat = "markdown"
```

### Step 3 — Match & Analyze

For each P (action item) in the OKR tree, determine its actual status by matching against collected data.

**Matching strategy**:
- Extract keywords from the P description (e.g., "签名问题", "signature", "抗量子", "post-quantum", "bug bounty", "TIP-PQ")
- Search for these keywords in:
  - GitHub PR titles and commit messages
  - Slack messages
  - Confluence weekly report content
- Assign status:

| Match Result | Status |
|-------------|--------|
| Column K = "是" | **Completed** |
| Column K = "否" | **Blocked / Not done** |
| Matched to merged PR or completed doc | **Completed** |
| Matched to open PR or in-progress work | **In progress** |
| Mentioned in Slack/weekly report recently | **In progress** |
| No match anywhere | **Not started** |

For each "in progress" or "completed" P, record the evidence (PR link, commit, doc link).

### Step 4 — Priority Calculation

```
quarter_start  = first day of current quarter
quarter_end    = last day of current quarter
quarter_days   = quarter_end - quarter_start + 1
quarter_progress = (today - quarter_start) / quarter_days

# Per KR
composite_weight = O_weight × KR_weight
kr_completion    = completed_P_count / total_P_count
behind_degree    = max(0, quarter_progress - kr_completion)
priority_score   = composite_weight × (1 + behind_degree × 2)
```

**Quarter boundaries**:
| Quarter | Start | End | Days |
|---------|-------|-----|------|
| Q1 | Jan 1 | Mar 31 | 90 |
| Q2 | Apr 1 | Jun 30 | 91 |
| Q3 | Jul 1 | Sep 30 | 92 |
| Q4 | Oct 1 | Dec 31 | 92 |

**Categorization** (by priority_score descending):

| Category | Criteria |
|----------|----------|
| High priority (this week) | Top 3 KRs by priority_score, or behind_degree > 0.2 |
| Medium priority (this month) | Middle tier, or behind_degree between 0.05 and 0.2 |
| Ongoing | The rest, or continuous tasks (e.g., bug bounty, on-duty) |

### Step 5 — Update OKR Completion Status

After analysis, write back completion marks to the xlsx file for P items that are determined to be completed based on external evidence (merged PRs, completed docs, etc.) but whose column K is still empty.

**Criteria for marking "是"**:
- The P's corresponding PR has been **merged** (not just open)
- The P's deliverable (doc, TIP, design) is **published** on Confluence
- The P is confirmed done in a **weekly report**

**Do NOT auto-mark** if:
- The PR is still open / in review
- The evidence only shows "in progress"
- Column K already has a value ("是" or "否")

**Implementation** using python3 + openpyxl:

```python
import openpyxl

wb = openpyxl.load_workbook('/Users/tron/federico/okr/2026-Federico-OKR.xlsx')
ws = wb['2026Q2']  # current quarter sheet

# updates is a list of (row_number, new_value) tuples
# e.g., [(4, '是'), (5, '是')]
for row, value in updates:
    ws.cell(row=row, column=11).value = value  # column K = 11

wb.save('/Users/tron/federico/okr/2026-Federico-OKR.xlsx')
```

**Important**: Before writing, present the proposed updates to the user and ask for confirmation:

```
The following P items appear completed based on evidence:
  Row 3  P1：完成签名问题分析、修复和测试 — PR#6652 merged
  Row 7  P5：完成zksnark-java-sdk 库代码优化 — weekly report confirmed

Mark these as "是" in the OKR file? (y/n)
```

Only write to the file after the user confirms.

### Step 6 — Terminal Output

Output plain text to the terminal. Format:

```
====================================================
  OKR Task Advisor — {year} Q{quarter}
  Quarter progress: Day {X}/{total} ({Y}%)
  Today: {date}
====================================================

--- High Priority (This Week) ---

1. [O{n}-KR{m}] {KR description}
   Weight: {composite_weight} | Completion: {done}/{total} P | Behind: {behind}%
   Completed: {P description} — {evidence link}
   In progress: {P description} — {evidence link}
   Not started: {P description}
   >>> Suggestion: {actionable recommendation}

2. [O{n}-KR{m}] {KR description}
   ...

--- Medium Priority (This Month) ---

3. [O{n}-KR{m}] {KR description}
   ...

--- Ongoing ---

4. [O{n}-KR{m}] {KR description}
   ...

====================================================
  Summary: {completed_KR}/{total_KR} KRs on track
  {high_count} items need attention this week
====================================================
```

**Output rules**:
- KR descriptions can be in Chinese (as they appear in the OKR file)
- Suggestions must be specific and actionable (not generic "keep going")
- Include evidence links (GitHub PR URLs, Confluence page URLs) where available
- For "not started" high-priority items, suggest a concrete first step
- For "in progress" items, suggest the next action (e.g., "complete review", "merge and start P2")
- For continuous tasks (bug bounty, on-duty), note recent activity and suggest maintaining pace

---

## Analysis Quality Guidelines

### DO
- Parse every P in the current quarter sheet — do not skip any
- Cross-reference with ALL data sources before marking a P as "not started"
- Include evidence links for every status determination
- Consider dependencies between Ps (e.g., P2 may depend on P1 completion)
- Weight suggestions by both composite_weight and behind_degree

### DON'T
- Mark a P as "completed" without evidence from column K or external data
- Ignore the weight system — a low-weight KR should not outrank a high-weight one
- Give vague suggestions like "continue working" — be specific about the next action
- Assume KRs from previous quarters carry forward — only analyze the current quarter sheet

---

## Data Source Reference

| Source | Method | Key Parameters |
|--------|--------|---------------|
| OKR file | python3 + openpyxl | `/Users/tron/federico/okr/2026-Federico-OKR.xlsx` |
| GitHub PRs (personal) | `gh api search/issues` | `author:Federico2014`, `type:pr`, `created:>=QUARTER_START` |
| GitHub PRs (org) | `gh api search/issues` | `author:Federico2014`, `org:tronprotocol`, `type:pr`, `updated:>=QUARTER_START` |
| GitHub reviews | `gh api search/issues` | `reviewed-by:Federico2014`, `org:tronprotocol`, `type:pr` |
| GitHub issues | `gh api search/issues` | `involves:Federico2014`, `org:tronprotocol`, `type:issue` |
| GitHub commits | `gh api search/commits` | `author:Federico2014`, `committer-date:>=QUARTER_START` |
| Slack messages | `curl` + Slack API | Channel `C0ARYHYLZ3Q`, `$SLACK_BOT_TOKEN` from mcp.json, `oldest` = 2 weeks ago |
| Confluence search | `searchConfluenceUsingCql` | `cloudId: "troneco.atlassian.net"`, space `javatronx` |
| Confluence read | `getConfluencePage` | `cloudId: "troneco.atlassian.net"`, `contentFormat: "markdown"` |
