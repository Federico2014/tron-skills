---
name: weekly-report
description: >
  Generate Federico's weekly work report for the javatronx Confluence space.
  Collects data from Slack (daily-ai-assistant channel), GitHub (Federico2014 and tronprotocol repos),
  and organizes into three tables: This Week's Accomplishments, Problems Encountered, Next Week's Plan.
  Output in Confluence Storage Format or Markdown.
  Trigger phrases: "write weekly report", "generate weekly report", "weekly report", "/weekly-report".
---

# weekly-report

Automatically collect weekly work data from multiple sources and generate a personal weekly report matching the javatronx Confluence template for Federico.

## Usage

```bash
/weekly-report
```

## User Identity

| Field | Value |
|-------|-------|
| Name | Federico |
| GitHub | Federico2014 (personal), tronprotocol (org) |
| Email | federico.zhen@tron.network |
| Confluence Space | javatronx |
| Slack Channel | `daily-ai-assistant` (ID: `C0ARYHYLZ3Q`) |

## Time Range

- Report covers: **last Friday 12:00** to **this Friday 12:00** (UTC+8)
- "This week" always refers to the above time range
- Deadline: every Friday before 12:00 noon

---

## Workflow

### Step 1 — Calculate Time Range

Automatically compute the report start and end dates based on the current date.

```bash
# macOS date calculation (example)
this_friday=$(date -v+Fri -j +%Y-%m-%d)
last_friday=$(date -v-7d -j -f "%Y-%m-%d" "$this_friday" +%Y-%m-%d)
range_start="${last_friday}T12:00:00+08:00"
range_end="${this_friday}T12:00:00+08:00"
echo "Report range: $range_start → $range_end"
echo "Page name: Federico - $(echo $this_friday | tr -d '-')"
```

- If today is Friday before 12:00, `this_friday` = today
- If today is Friday after 12:00, `this_friday` = next Friday
- Page naming: `Federico - YYYYMMDD`

### Step 2 — Collect Data from All Sources

Collect from all sources in parallel. Filter all results by time range.

#### 2a. Slack: daily-ai-assistant Daily Notes

Read daily work notes using Slack MCP tools:

```
Tool: slack_get_channel_history
Params: channel_id = "C0ARYHYLZ3Q", limit = 200
```

**Important**: `slack_get_channel_history` does not support `oldest`/`latest` parameters. Fetch messages in bulk and filter by `ts` (Unix timestamp) to keep only messages within the time range.

For messages with threads, use `slack_get_thread_replies` to get full context:
```
Tool: slack_get_thread_replies
Params: channel_id = "C0ARYHYLZ3Q", thread_ts = <parent message ts>
```

#### 2b. GitHub: Personal Repos (Federico2014)

```bash
# Commits in date range
gh api "/search/commits?q=author:Federico2014+committer-date:${last_friday}..${this_friday}&sort=committer-date&per_page=100" \
  --jq '.items[] | {repo: .repository.full_name, sha: .sha[0:7], message: .commit.message, date: .commit.committer.date, url: .html_url}'

# PRs created
gh api "/search/issues?q=author:Federico2014+type:pr+created:${last_friday}..${this_friday}&per_page=100" \
  --jq '.items[] | {title: .title, number: .number, state: .state, url: .html_url, repo: (.repository_url | split("/") | .[-1])}'
```

#### 2c. GitHub: Organization Repos (tronprotocol)

```bash
# PRs authored (created or updated in range)
gh api "/search/issues?q=author:Federico2014+org:tronprotocol+type:pr+updated:${last_friday}..${this_friday}&per_page=100" \
  --jq '.items[] | {title: .title, number: .number, state: .state, url: .html_url, repo: (.repository_url | split("/") | .[-1])}'

# PRs reviewed
gh api "/search/issues?q=reviewed-by:Federico2014+org:tronprotocol+type:pr+updated:${last_friday}..${this_friday}&per_page=100" \
  --jq '.items[] | {title: .title, number: .number, url: .html_url, repo: (.repository_url | split("/") | .[-1])}'

# Issues commented on
gh api "/search/issues?q=commenter:Federico2014+org:tronprotocol+updated:${last_friday}..${this_friday}&per_page=100" \
  --jq '.items[] | {title: .title, number: .number, url: .html_url, repo: (.repository_url | split("/") | .[-1])}'
```

#### 2d. Present Raw Data Summary

After collection, present a structured summary organized by source:
- Slack daily notes summary
- GitHub PRs (authored / reviewed)
- GitHub Issues (commented)
- GitHub Commits

### Step 3 — User Supplements

Ask the user two questions:

1. "Is anything missing from the data above? Any meetings, research, documentation, ad-hoc tasks, or collaboration items to add?"
2. "Did you encounter any problems this week (technical / resource / collaboration)?"

Wait for user response before proceeding.

### Step 4 — Generate Report

Organize all collected and supplemented data into three tables. The report content (table headers, categories, status values) MUST be in **Chinese** as required by the Confluence template.

#### Table 1: 本周完成 (This Week's Accomplishments)

| 任务类别 | 进展明细 | 进展状态 |
|---------|---------|---------|
| 项目相关 | **Sub-project name**<br/>- Specific work + PR/Issue links<br/>- Specific work + doc links | Status |
| 日常工作 | Issue/PR Review, Bug Bounty, AI workflow, etc. | Status |
| 临时任务 | Ad-hoc work items | Status |

**Classification rules**:
- **项目相关** (Project-related): Group by sub-project name (bolded). List specific work items under each sub-project with PR/Issue/doc links. Status should be precise per sub-project: `已完成`, `进行中`, `review中`, `待合并`, `CI 通过，待 review`, etc.
- **日常工作** (Routine work): Code reviews (list each reviewed PR), issue analysis, bug bounty, AI workflow, hiring, etc.
- **临时任务** (Ad-hoc tasks): On-duty rotation, interviews, meetings, and other non-routine items
- Omit categories with no content

#### Table 2: 遇到的问题 (Problems Encountered)

| 问题类别 | 问题描述以及影响 | 需求 |
|---------|----------------|------|
| 技术类（方案、性能、缺陷、依赖、质量）| Problem + impact | 已修复/已规避/待解决 |
| 资源类（权限、环境、数据、预算、设备）| | |
| 协作类（跨团队、需求变更、审批）| | |

**Writing rules**:
- Each problem must state: what happened -> what the impact was -> current resolution (已修复/已规避/待解决)
- Omit categories with no problems
- If no problems this week, fill with "本周无"

#### Table 3: 下周计划 (Next Week's Plan)

| 任务类型 | 计划简述 | 优先级 |
|---------|---------|-------|
| 项目相关 | Project name + planned work | 高/中/低 |
| 日常工作 | Ongoing routine work | Priority |
| 临时任务 | Known upcoming ad-hoc items | Priority |
| 长期任务 | Long-term research/improvement items | Priority |

**Derivation rules**:
- Items from Table 1 with status other than `已完成` automatically carry forward to next week's plan
- In-progress PR reviews carry forward as "跟进 PR #N review 反馈"
- Merge in any additional plans the user provides
- Omit categories with no content

### Step 5 — User Review

Present the complete draft report and ask the user to review. Apply any changes until the user confirms.

### Step 6 — Publish to Confluence

After the user confirms the report, publish it to the corresponding Confluence page.

**Find the target page**: Search for the page named `Federico - YYYYMMDD` in the javatronx space.

```
Tool: searchConfluenceUsingCql
Params: cloudId = "troneco.atlassian.net"
        cql = "space = 'javatronx' AND title = 'Federico - YYYYMMDD' AND type = page"
```

**Update the page** with the generated report content:

```
Tool: updateConfluencePage
Params: cloudId = "troneco.atlassian.net"
        pageId = <page ID from search result>
        body = <generated report in markdown>
        contentFormat = "markdown"
        versionMessage = "Weekly report auto-generated"
```

If the page is not found, inform the user and output the report content for manual paste instead.

---

## Output Format

### Primary: Confluence Storage Format (XHTML)

```html
<h2>本周完成</h2>
<table data-layout="default">
<thead>
<tr><th>任务类别</th><th>进展明细</th><th>进展状态</th></tr>
</thead>
<tbody>
<tr>
<td>项目相关</td>
<td>
<p><strong>子项目名</strong></p>
<ul>
<li>具体工作内容 <a href="https://github.com/Federico2014/java-tron/pull/9">PR #9</a></li>
<li>具体工作内容 <a href="https://github.com/Federico2014/java-tron/pull/10">PR #10</a></li>
</ul>
</td>
<td>PR #9 review 中<br/>PR #10 CI 通过，待 review</td>
</tr>
</tbody>
</table>

<h2>遇到的问题</h2>
<table data-layout="default">
<thead>
<tr><th>问题类别</th><th>问题描述以及影响</th><th>需求</th></tr>
</thead>
<tbody>
<tr><td>技术类（方案、性能、缺陷、依赖、质量）</td><td>问题描述</td><td>已修复</td></tr>
</tbody>
</table>

<h2>下周计划</h2>
<table data-layout="default">
<thead>
<tr><th>任务类型</th><th>计划简述</th><th>优先级</th></tr>
</thead>
<tbody>
<tr><td>项目相关</td><td>计划内容</td><td>高</td></tr>
</tbody>
</table>
```

### Fallback: Markdown Tables

Use standard Markdown tables when Confluence is not available, for preview or manual paste.

---

## Output Rules

- **Language**: All table content, headers, categories, and status values MUST be in Chinese (matching the Confluence template)
- **Links**: GitHub PR/Issue must include full URLs as hyperlinks
- **Grouping**: Under 项目相关, group work by sub-project name (bolded), with bullet-pointed detail items beneath
- **Status precision**: Match each sub-project's actual state — do not use generic "进行中" when a more specific status like "PR #9 review 中" applies
- **Auto carry-forward**: Incomplete items from Table 1 automatically appear in Table 3
- **Empty categories**: Omit from the table rather than leaving empty rows
- **Page naming**: `Federico - YYYYMMDD` (this Friday's date)

---

## Data Source Reference

| Source | Tool | Key Parameters |
|--------|------|---------------|
| Slack daily notes | `slack_get_channel_history` | `channel_id: "C0ARYHYLZ3Q"`, `limit: 200` |
| Slack threads | `slack_get_thread_replies` | `channel_id`, `thread_ts` |
| GitHub commits | `gh api /search/commits` | `author:Federico2014`, `committer-date:RANGE` |
| GitHub PRs (personal) | `gh api /search/issues` | `author:Federico2014`, `type:pr`, `created:RANGE` |
| GitHub PRs (org) | `gh api /search/issues` | `author:Federico2014`, `org:tronprotocol`, `type:pr`, `updated:RANGE` |
| GitHub reviews | `gh api /search/issues` | `reviewed-by:Federico2014`, `org:tronprotocol`, `type:pr`, `updated:RANGE` |
| GitHub issues | `gh api /search/issues` | `commenter:Federico2014`, `org:tronprotocol`, `updated:RANGE` |
| Confluence (read) | `getConfluencePage` | `cloudId: "troneco.atlassian.net"`, `pageId`, `contentFormat: "markdown"` |
| Confluence (search) | `searchConfluenceUsingCql` | `cloudId: "troneco.atlassian.net"`, `cql: "space = 'javatronx' AND title = '...'"` |
| Confluence (write) | `updateConfluencePage` | `cloudId: "troneco.atlassian.net"`, `pageId`, `contentFormat: "markdown"` |

---

## Limitations

- `slack_get_channel_history` does not support `oldest`/`latest` time filtering; messages are fetched in bulk and filtered by timestamp
- `slack_search_messages` is not available in the current Slack MCP server; workspace-wide search is skipped
