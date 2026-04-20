---
name: weekly-report
description: >
  Generate Federico's weekly work report for the javatronx Confluence space.
  Collects data from Slack (daily-ai-assistant channel), GitHub (Federico2014 and tronprotocol repos),
  and organizes into three tables: This Week's Accomplishments, Problems Encountered, Next Week's Plan.
  Output in ADF (Atlassian Document Format) with grey header backgrounds, ordered+nested bullet lists, and precise per-item status alignment.
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

> **Time zone note**: GitHub API dates are in UTC. The report range (last Friday 12:00 → this Friday 12:00 UTC+8) maps to UTC as `${last_friday}T04:00:00Z` → `${this_friday}T04:00:00Z`. Use these UTC bounds for all GitHub API date filtering.

#### 2a. Slack: All Activity via Official Slack MCP

使用官方 Slack MCP（`mcp.slack.com`，OAuth 2.1），可访问所有频道（公开 + 私有 + DM），无需 Bot 邀请。

**2a-i. 搜索本周 Federico 发出的消息（覆盖所有频道）**

```
Tool: slack_search_public_and_private
Params:
  query = "from:me after:${last_friday_date} before:${this_friday_date}"
  sort = "timestamp"
  sort_dir = "asc"
  limit = 20
```

若结果有下一页（`pagination_info` 含 cursor），继续翻页直到取完。

**2a-ii. 搜索本周发给 Federico 的消息（DM + @mention）**

```
Tool: slack_search_public_and_private
Params:
  query = "to:me after:${last_friday_date} before:${this_friday_date}"
  sort = "timestamp"
  sort_dir = "asc"
  limit = 20
```

**2a-iii. 读取 daily-ai-assistant 频道完整历史（含所有人发言）**

```
Tool: slack_read_channel
Params:
  channel_id = "C0ARYHYLZ3Q"
  oldest = "${last_friday_unix_ts}"   # last_friday 12:00 UTC+8 的 Unix 时间戳
  latest = "${this_friday_unix_ts}"   # this_friday 12:00 UTC+8 的 Unix 时间戳
  limit = 100
```

`oldest`/`latest` 时间戳换算：
```bash
last_friday_unix=$(date -j -f "%Y-%m-%d %H:%M:%S" "${last_friday} 12:00:00" +%s)
this_friday_unix=$(date -j -f "%Y-%m-%d %H:%M:%S" "${this_friday} 12:00:00" +%s)
```

**2a-iv. 读取重要消息的线程回复**

```
Tool: slack_read_thread
Params:
  channel_id = <channel_id>
  message_ts = <parent message ts>
```

**关键频道列表**：

| Channel | ID | 用途 |
|---------|-----|------|
| `daily-ai-assistant` | `C0ARYHYLZ3Q` | AI 工作日报、文档分享 |
| `search-task` | `C0ADF3TJQRE` | 搜索/后量子签名方案讨论 |

#### 2b. GitHub: All Activity via Events API

**用 events API 代替 search API**——events API 覆盖所有仓库（个人 fork + 个人仓库 + org 仓库），`org:tronprotocol` 过滤会遗漏个人 fork 中的开发活动。

```bash
# 获取 Federico2014 所有公开事件（最近 ~90 天，最多 300 条）
gh api "/users/Federico2014/events?per_page=100" | \
  jq --arg s "${last_friday}T04:00:00Z" --arg e "${this_friday}T04:00:00Z" \
  '[.[] | select(.created_at >= $s and .created_at <= $e)]' > /tmp/gh_events.json

# 统计事件类型分布（快速概览）
jq 'group_by(.type) | .[] | {type: .[0].type, repos: [.[].repo.name] | unique, count: length}' /tmp/gh_events.json
```

从缓存文件分别提取各类活动：

```bash
# PRs 创建/合并（含个人 fork 和 org）
jq '.[] | select(.type == "PullRequestEvent") |
  {action: .payload.action, number: .payload.pull_request.number,
   repo: .repo.name, created_at: .created_at}' /tmp/gh_events.json

# PR Reviews 提交（含个人 fork 和 org）
jq '.[] | select(.type == "PullRequestReviewEvent") |
  {pr: .payload.pull_request.number, repo: .repo.name,
   state: .payload.review.state, submitted_at: .payload.review.submitted_at,
   body: (.payload.review.body // "" | .[0:150])}' /tmp/gh_events.json

# Issue/PR 评论
jq '.[] | select(.type == "IssueCommentEvent") |
  {number: .payload.issue.number, repo: .repo.name,
   created_at: .created_at, url: .payload.comment.html_url,
   body: (.payload.comment.body | .[0:150])}' /tmp/gh_events.json

# Commits pushed
jq '.[] | select(.type == "PushEvent") |
  {repo: .repo.name, ref: .payload.ref, created_at: .created_at,
   commits: [.payload.commits[]?.message | split("\n")[0]]}' /tmp/gh_events.json
```

**获取 PR 详情**（events 中 PR title 有时为 null，需单独请求）：
```bash
gh api "/repos/{owner}/{repo}/pulls/{number}" \
  --jq '{title: .title, url: .html_url, body: (.body | .[0:300])}'
```

#### 2c. GitHub: Review & Comment 详情补全

Events API 提供的 review/comment 内容有时被截断，需补充拉取完整内容及精确链接：

**PR review 内联评论**（inline comments 不在 PullRequestReviewEvent 中，需单独拉）：
```bash
gh api "/repos/{owner}/{repo}/pulls/{number}/comments" | \
  jq --arg s "${last_friday}T04:00:00Z" --arg e "${this_friday}T04:00:00Z" \
  '.[] | select(.user.login == "Federico2014" and .created_at >= $s and .created_at <= $e) |
   {url: .html_url, body: (.body | .[0:200])}'
```

Review 直链格式：`https://github.com/{owner}/{repo}/pull/{number}#pullrequestreview-{review_id}`

**Issue 评论完整内容**（events 中 body 已有，但如需完整内容）：
```bash
gh api "/repos/{owner}/{repo}/issues/{number}/comments" | \
  jq --arg s "${last_friday}T04:00:00Z" --arg e "${this_friday}T04:00:00Z" \
  '.[] | select(.user.login == "Federico2014" and .created_at >= $s and .created_at <= $e) |
   {url: .html_url, body: (.body | .[0:300])}'
```

`html_url` 即直达评论链接。

> **`gh api` + `jq --arg` 注意**：`gh api --jq` 不支持 `--arg`，必须 pipe 给 `jq`：`gh api "..." | jq --arg key val '...'`

#### 2d. Confluence: Pages Created or Updated

```
Tool: searchConfluenceUsingCql
Params: cloudId = "troneco.atlassian.net"
        cql = "space = 'javatronx' AND contributor = '62b993a8fbc1f7c647b76104' AND lastModified >= '${last_friday_date}' ORDER BY lastModified DESC"
        limit = 50
```

`${last_friday_date}` 只取日期部分（如 `2026-04-11`）。

**重要**：CQL `contributor`/`creator` 字段必须使用 Atlassian account ID，不能用邮箱。Federico 的 account ID 为 `62b993a8fbc1f7c647b76104`。

同时执行两条查询并去重：
1. **新建页面**：`space = 'javatronx' AND creator = '62b993a8fbc1f7c647b76104' AND created >= '${last_friday_date}'`
2. **更新页面**：上方查询（涵盖本周参与编辑的所有页面）

两条查询都返回的 → 新建；仅在查询 2 中的 → 更新。

每条结果提取：`title`、`id`、`_links.webui`（页面 URL）、`lastModified`（更新时间）。

#### 2e. Present Raw Data Summary

采集完成后，按来源输出结构化摘要供下一步使用：
- Slack 本周消息摘要
- GitHub PRs（authored / reviewed）
- GitHub Issues（commented）
- GitHub Commits（pushed）
- Confluence 页面（新建 / 更新）

### Step 3 — User Supplements

Ask the user two questions:

1. "Is anything missing from the data above? Any meetings, research, documentation, ad-hoc tasks, or collaboration items to add?"
2. "Did you encounter any problems this week (technical / resource / collaboration)?"

Wait for user response before proceeding.

### Step 4 — Generate Report

Organize all collected and supplemented data into three tables using **ADF (Atlassian Document Format)**. All content MUST be in **Chinese**.

#### ADF Table Structure Rules

- Table `layout`: `"center"`, `width`: 882 (本周完成/下周计划) or 892 (遇到的问题)
- Header row cells: `"background": "#f0f1f2"`, text **bold**
- Data cells: `"background": "#ffffff"`
- Column widths for 本周完成: `[136, 560, 185]`
- Column widths for 遇到的问题: `[183, 411, 297]`
- Column widths for 下周计划: `[179, 409, 294]`

#### Cell Content Structure

**进展明细 / 计划简述 cells** — use `orderedList` at top level, each item has:
- A `paragraph` with the sub-project/category name in **bold**
- A nested `bulletList` with detail lines (links inline)

**进展状态 / 优先级 cells** — use `orderedList` with one item per top-level item in the adjacent cell (numbered to match)

**All text must be Chinese**. PR/Issue/doc references are links embedded inline in text.

**描述语言规则**：所有工作描述必须用简洁中文概括实际贡献，禁止直接使用英文 PR 标题、commit message 或英文技术词组作为描述文本。PR/Issue 编号和链接可保留，但链接前后的描述文字必须是中文。例如：
- ✅ `修复节点同步时的内存泄漏问题，` [PR #123](url)
- ❌ `Fix memory leak in node sync` [PR #123](url)

#### Table 1: 本周完成（M月D日 — M月D日）

Heading includes the date range: `本周完成（4月10日 — 4月17日）`

Row categories (omit rows with no content):
- **项目相关**: One numbered item per sub-project (bold title in Chinese + bullet details in Chinese + PR/doc links). Each bullet must describe the contribution in Chinese, not copy the PR/commit title.
- **日常工作**: Numbered items for PR Review / Issue分析 / 漏洞赏金 / AI工作流 / 文档 etc., each with bullet sub-items:
  - **PR Review**: One bullet per reviewed PR. Link text = `PR #N`，前置中文描述说明该 PR 的主要内容，后跟直达 review 链接（`#pullrequestreview-{id}`），再附 1 句中文说明 review 的主要意见或结论。
  - **Issue分析**: One bullet per issue commented on. Link text = `#N`，前置中文描述说明该 Issue 的主题，后跟直达评论链接（`html_url`），再附 1 句中文说明评论要点。
  - Only include reviews/comments whose timestamp falls within the report time range (verified in Step 2c).
  - **文档**: One bullet per Confluence page created or updated (from Step 2e). Link to the page; indicate whether 新建 or 更新，加 1 句中文说明文档主要内容。若文档属于某个子项目，归入 项目相关 而非此处。
- **临时任务**: Ad-hoc tasks described in Chinese (omit if empty)

#### Table 2: 遇到的问题

Only include rows that have actual content. If no problems: one row with `技术类（方案、性能、缺陷、依赖、质量）` and `本周无` in the description cell. Omit 资源类 and 协作类 rows if empty.

Problem format (when problems exist):
- **Bold** problem title, then bullet list: what happened → impact → resolution (已修复/已规避/待解决)

#### Table 3: 下周计划

Row categories (omit if no content):
- **项目相关**: Carry forward unfinished items from Table 1; one numbered item per sub-project
- **日常工作**: Ongoing routine work
- **临时任务**: Known upcoming ad-hoc (omit if empty)
- **长期任务**: Long-term research/improvement items

**Derivation rules**:
- Items from Table 1 with status other than `已完成` automatically carry forward
- In-progress PR reviews carry forward as "跟进 PR #N review 反馈"
- Merge in any additional plans the user provides

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
        body = <generated report as ADF JSON string>
        contentFormat = "adf"
        versionMessage = "Weekly report auto-generated"
```

If the page is not found, inform the user and output the report content for manual paste instead.

---

## Output Format

### Primary: ADF JSON (Atlassian Document Format)

Use `contentFormat: "adf"` when calling `updateConfluencePage`. The body must be a valid ADF JSON string.

**Skeleton structure**:

```json
{
  "type": "doc",
  "version": 1,
  "content": [
    {"type": "heading", "attrs": {"level": 2}, "content": [{"type": "text", "text": "本周完成（M月D日 — M月D日）"}]},
    {
      "type": "table",
      "attrs": {"layout": "center", "width": 882},
      "content": [
        {
          "type": "tableRow",
          "content": [
            {"type": "tableHeader", "attrs": {"colspan": 1, "rowspan": 1, "background": "#f0f1f2", "colwidth": [136]},
             "content": [{"type": "paragraph", "content": [{"type": "text", "text": "任务类别", "marks": [{"type": "strong"}]}]}]},
            {"type": "tableHeader", "attrs": {"colspan": 1, "rowspan": 1, "background": "#f0f1f2", "colwidth": [560]},
             "content": [{"type": "paragraph", "content": [{"type": "text", "text": "进展明细", "marks": [{"type": "strong"}]}]}]},
            {"type": "tableHeader", "attrs": {"colspan": 1, "rowspan": 1, "background": "#f0f1f2", "colwidth": [185]},
             "content": [{"type": "paragraph", "content": [{"type": "text", "text": "进展状态", "marks": [{"type": "strong"}]}]}]}
          ]
        },
        {
          "type": "tableRow",
          "content": [
            {"type": "tableCell", "attrs": {"colspan": 1, "rowspan": 1, "background": "#ffffff", "colwidth": [136]},
             "content": [{"type": "paragraph", "content": [{"type": "text", "text": "项目相关"}]}]},
            {
              "type": "tableCell", "attrs": {"colspan": 1, "rowspan": 1, "background": "#ffffff", "colwidth": [560]},
              "content": [{
                "type": "orderedList", "attrs": {"order": 1},
                "content": [{
                  "type": "listItem",
                  "content": [
                    {"type": "paragraph", "content": [{"type": "text", "text": "子项目名", "marks": [{"type": "strong"}]}]},
                    {"type": "bulletList", "content": [
                      {"type": "listItem", "content": [{"type": "paragraph", "content": [
                        {"type": "text", "text": "具体工作内容 "},
                        {"type": "text", "text": "PR #9", "marks": [{"type": "link", "attrs": {"href": "https://github.com/..."}}]}
                      ]}]}
                    ]}
                  ]
                }]
              }]
            },
            {
              "type": "tableCell", "attrs": {"colspan": 1, "rowspan": 1, "background": "#ffffff", "colwidth": [185]},
              "content": [{"type": "orderedList", "attrs": {"order": 1}, "content": [
                {"type": "listItem", "content": [{"type": "paragraph", "content": [{"type": "text", "text": "PR #9 review 中"}]}]}
              ]}]
            }
          ]
        }
      ]
    },
    {"type": "heading", "attrs": {"level": 2}, "content": [{"type": "text", "text": "遇到的问题"}]},
    {"type": "table", "attrs": {"layout": "center", "width": 892}, "content": ["... colwidths: [183, 411, 297] ..."]},
    {"type": "heading", "attrs": {"level": 2}, "content": [{"type": "text", "text": "下周计划"}]},
    {"type": "table", "attrs": {"layout": "center", "width": 884}, "content": ["... colwidths: [179, 409, 294] ..."]}
  ]
}
```

### Fallback: Markdown Tables

Use standard Markdown tables only when Confluence is unavailable (preview or manual paste).

---

## Output Rules

- **Language**: All content MUST be in Chinese. PR/Issue numbers, repo names, and hyperlink hrefs are the only English-allowed elements. Never copy English PR titles, commit messages, or technical phrases as-is — always translate/summarize into concise Chinese descriptions.
- **Format**: Always use ADF (`contentFormat: "adf"`) when publishing to Confluence
- **Links**: Embed GitHub PR/Issue/doc links inline within text nodes using ADF link marks
- **Grouping**: Use `orderedList` → `listItem` (bold title) → nested `bulletList` for each sub-project/category
- **Status alignment**: 进展状态 / 优先级 column uses `orderedList` with items numbered to match the adjacent column
- **Status precision**: Use specific status like "PR #9 review 中" not generic "进行中"
- **Auto carry-forward**: Incomplete items from Table 1 automatically appear in Table 3
- **Empty rows**: Omit rather than leaving blank (exception: 技术类 row always present in 遇到的问题)
- **Page naming**: `Federico - YYYYMMDD` (this Friday's date)

---

## Data Source Reference

| Source | Tool / API | Key Parameters |
|--------|-----------|---------------|
| Slack 全渠道搜索（发出） | `slack_search_public_and_private` | `query: "from:me after:YYYY-MM-DD before:YYYY-MM-DD"` |
| Slack 全渠道搜索（收到） | `slack_search_public_and_private` | `query: "to:me after:YYYY-MM-DD before:YYYY-MM-DD"` |
| Slack 频道历史 | `slack_read_channel` | `channel_id`, `oldest`/`latest`（Unix 时间戳），`limit: 100` |
| Slack 线程回复 | `slack_read_thread` | `channel_id`, `message_ts` |
| GitHub 全量活动 | `gh api /users/Federico2014/events` | `per_page=100`；覆盖所有仓库（personal + org）；日期用 UTC |
| GitHub PR 详情 | `gh api /repos/{owner}/{repo}/pulls/{number}` | 补全 events 中 null 的 title/body |
| GitHub review 内联评论 | `gh api /repos/{owner}/{repo}/pulls/{number}/comments` | 过滤 `user.login == "Federico2014"` 和 `created_at` 在 UTC 范围内 |
| GitHub issue 评论 | `gh api /repos/{owner}/{repo}/issues/{number}/comments` | 过滤 `user.login == "Federico2014"` 和 `created_at` 在 UTC 范围内；`html_url` 为直达链接 |
| Confluence 活动 | `searchConfluenceUsingCql` | `space='javatronx' AND contributor='62b993a8fbc1f7c647b76104' AND lastModified>='YYYY-MM-DD'`；必须用 account ID |
| Confluence 读取 | `getConfluencePage` | `cloudId: "troneco.atlassian.net"`, `pageId`, `contentFormat: "markdown"` |
| Confluence 搜索 | `searchConfluenceUsingCql` | `space='javatronx' AND title='Federico - YYYYMMDD'` |
| Confluence 写入 | `updateConfluencePage` | `cloudId: "troneco.atlassian.net"`, `pageId`, `contentFormat: "adf"` |

**固定值**：
- Federico 的 Atlassian account ID：`62b993a8fbc1f7c647b76104`
- Slack `daily-ai-assistant` channel ID：`C0ARYHYLZ3Q`
- Slack `search-task` channel ID：`C0ADF3TJQRE`

---

## Limitations

- Slack 官方 MCP (`mcp.slack.com`) 需工作区管理员审批并完成 OAuth 授权后才可使用；未授权时退回 `daily-ai-assistant` 频道的 `slack_read_channel` 调用
- `slack_search_public_and_private` 每页最多 20 条，需按 cursor 翻页；`slack_read_channel` 每页最多 100 条
- `gh api /users/{user}/events` 最多返回 300 条，覆盖约 90 天；超出范围的历史活动需用 search API 补查
- `gh api --jq` 不支持 `--arg`，需 pipe 给 `jq`：`gh api "..." | jq --arg key val '...'`
- GitHub events API 中 PR/Issue title 有时为 null，需单独调用 pulls/{number} 补全
