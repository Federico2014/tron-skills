---
name: bugbounty
description: >
  Review HackerOne bug bounty reports against the java-tron codebase.
  Fetches the report via HackerOne MCP, analyzes the vulnerability against java-tron develop branch,
  determines validity, and publishes a structured analysis page to Confluence under java-tron BugBounty.
  Trigger phrases: "review bug bounty", "analyze hackerone report", "bug bounty review", "/bugbounty".
---

# bugbounty

Review HackerOne bug bounty reports for java-tron. Fetch report details, analyze the reported vulnerability against the actual codebase, assess validity, and publish a structured analysis to Confluence.

## Usage

```bash
/bugbounty
```

The user provides a HackerOne report number (e.g., `3628180`). The skill fetches the report, analyzes the vulnerability against java-tron code, and creates a Confluence analysis page.

---

## Prerequisites

### HackerOne MCP Server

One of the following HackerOne MCP servers must be configured:

**Option A — Official GraphQL MCP (Docker, recommended)**

```bash
claude mcp add hackerone \
  -- docker run -i --rm \
  -e ENDPOINT="https://hackerone.com/graphql" \
  -e TOKEN="<base64(username:api_key)>" \
  -e MUTATION_MODE="none" \
  hackertwo/hackerone-graphql-mcp-server:1.0.7
```

Generate token: `echo -n "username:api_key" | base64`
Get API key at: https://hackerone.com/settings/api_token/edit

**Option B — Community REST API MCP (Node.js)**

```bash
git clone https://github.com/Sicks3c/hackerone-mcp-server.git
cd hackerone-mcp-server && npm install && npm run build

claude mcp add hackerone \
  -e H1_USERNAME=your-username \
  -e H1_API_TOKEN=your-api-token \
  -s user \
  -- node /path/to/dist/index.js
```

### Atlassian MCP Server

Required for publishing to Confluence. Should already be configured:

```json
{
  "atlassian": {
    "command": "npx",
    "args": ["-y", "mcp-remote@latest", "https://mcp.atlassian.com/v1/sse"]
  }
}
```

### java-tron Repository

The `gh` CLI must be authenticated with access to `tronprotocol/java-tron`.

---

## Workflow

### Step 1 — Parse Input

Extract the HackerOne report number from user input.

**Accepted input formats**:
- Report number: `3628180`
- HackerOne URL: `https://hackerone.com/reports/3628180`
- Email subject: `[Bug Bounty][#3628180] ...`

```
report_number = extract numeric ID from input
report_url = "https://hackerone.com/reports/{report_number}"
```

### Step 2 — Fetch Report from HackerOne

Use HackerOne MCP to retrieve the full report.

**With GraphQL MCP**: Query for the report by ID, requesting title, vulnerability_information, severity, weakness, structured_scope, and reporter details.

**With REST API MCP**:
```
Tool: get_report
Params: report_id = <report_number>
```

Extract and organize:
- **Title**: Report title / vulnerability summary
- **Severity**: Critical / High / Medium / Low
- **Weakness type**: CWE category (e.g., CWE-400 DoS, CWE-119 Buffer Overflow)
- **Vulnerability details**: Full description from the reporter
- **PoC / Steps to reproduce**: If provided
- **Affected component**: Module, endpoint, or function mentioned
- **Structured scope**: Target asset info

If the HackerOne MCP is not available, ask the user to paste the report content manually.

### Step 3 — Locate Affected Code in java-tron

Based on the report details, locate the relevant code in the java-tron develop branch.

```bash
# Clone or fetch latest develop branch info
gh api repos/tronprotocol/java-tron/git/ref/heads/develop \
  --jq '.object.sha'

# Search for the affected function/class/endpoint
gh api "search/code?q={function_or_class}+repo:tronprotocol/java-tron" \
  --jq '.items[] | {path: .path, url: .html_url}'
```

For deeper analysis, read the actual source files:

```bash
# Fetch file content from develop branch
gh api "repos/tronprotocol/java-tron/contents/{file_path}?ref=develop" \
  --jq '.content' | base64 -d
```

### Step 4 — Analyze Vulnerability

Perform a thorough analysis following this checklist:

#### 4a. Code Path Validation
- Locate the exact function/method mentioned in the report
- Trace all callers of the affected code (use grep/search across the codebase)
- For each caller, check:
  - Is there input validation before the call?
  - Is there exception handling wrapping the call?
  - Is this code path reachable from external input (API, P2P, RPC)?

#### 4b. PoC Validation
- If PoC is provided: Can the described attack vector actually reach the vulnerable code?
- If no PoC: Is the vulnerability theoretically exploitable?
- Check if preconditions for exploitation are realistic

#### 4c. Impact Assessment
- **DoS**: Can it crash the node or cause resource exhaustion?
- **Data integrity**: Can it corrupt blockchain state?
- **Confidentiality**: Can it leak sensitive data?
- **Authentication bypass**: Can it bypass access controls?
- Assess severity: Does the actual impact match the reporter's claimed severity?

#### 4d. Existing Mitigations
- Are there already guards in place (input validation, rate limiting, exception handlers)?
- Is the affected module even enabled in production? (e.g., PBFT module is disabled by default)
- Has this been fixed in a newer dependency version?

#### 4e. Conclusion

Reach one of these verdicts:

| Verdict | Meaning |
|---------|---------|
| **Valid (Critical/High/Medium/Low)** | Vulnerability confirmed, exploitable, with assessed severity |
| **Valid but Low Impact** | Vulnerability exists but mitigations reduce real-world impact |
| **Partially Valid** | Issue exists but is less severe than reported (e.g., no crash, only exception logged) |
| **Invalid** | Code path is not reachable, PoC is incorrect, or issue does not exist |
| **Informational** | Code quality issue worth fixing but not a security vulnerability |
| **Already Fixed** | Issue exists in the reported version but is fixed in current develop |

### Step 5 — Draft Analysis Report

Generate the analysis in the following structured format (in Chinese, matching Confluence template):

```markdown
## 问题描述

{Concise description of the reported vulnerability, including:
- What the reporter claims
- Affected component/function
- Claimed impact}

## 报告链接

HackerOne Report: https://hackerone.com/reports/{report_number}

## 严重性

- 报告者评级: {reporter's severity}
- 实际评估: {our assessed severity}

## 代码分析

{Detailed analysis following the structure from Step 4:

### 调用链分析
For each caller of the affected code:
(1) Caller name and location
- Code snippet showing the call context
- Whether input validation or exception handling exists
- Whether this path is reachable from external input

### PoC 验证
- Is the PoC valid?
- Can the attack vector reach the vulnerable code?

### 影响评估
- Actual impact vs. claimed impact
- Existing mitigations that reduce risk}

## 分析结论

**结论: {Verdict}**

{Summary reasoning: why the vulnerability is/isn't valid, what mitigations exist,
what the actual risk level is}

## 修复建议

{If the issue warrants a fix:
- Recommended code changes with code snippets
- Which files need modification
- Whether this is urgent or can be scheduled

If no fix needed:
- Explain why current code is sufficient
- Suggest optional code hardening if applicable}
```

### Step 6 — User Review

Present the complete analysis draft to the user. Wait for review and incorporate any changes.

Ask specifically:
1. "Does the analysis accurately reflect the code behavior?"
2. "Any adjustments to the verdict or severity assessment?"
3. "Ready to publish to Confluence?"

### Step 7 — Publish to Confluence

After user confirmation, create a new child page under the java-tron BugBounty parent page.

**Page title**: Use the report title or a descriptive summary (e.g., `getBase64FromByteString() 越界问题`)

```
Tool: createConfluencePage
Params:
  cloudId = "troneco.atlassian.net"
  spaceId = "751403098"
  parentId = "2071822585"
  title = "<descriptive title>"
  body = <generated analysis in markdown>
  contentFormat = "markdown"
  status = "current"
```

**Confluence page hierarchy**:
```
java-tron BugBounty 分析 (ID: 2071822585)
├── getBase64FromByteString() 越界问题
├── SM2 ASN1InputStream序列化问题
└── <new analysis page>   ← created by this skill
```

After publishing, return the page URL to the user.

---

## Analysis Quality Guidelines

### DO
- Trace ALL callers of the affected function, not just the obvious ones
- Show actual code snippets from java-tron develop branch to support conclusions
- Check if the affected module is even enabled in production
- Verify dependency versions (e.g., Bouncy Castle version for crypto issues)
- Consider both direct and indirect attack vectors
- Acknowledge valid concerns even when the overall verdict is "invalid"

### DON'T
- Dismiss a report without thorough code analysis
- Accept the reporter's severity rating without independent assessment
- Ignore edge cases in the call chain
- Skip checking exception handling around vulnerable calls
- Assume a function is unused without verifying with code search

### Code Analysis Patterns

**For DoS vulnerabilities**:
1. Can the attacker send input that triggers the issue via P2P or API?
2. Is there rate limiting or input size validation?
3. Does the exception crash the node or just log an error?
4. Is the affected code on the critical path (block processing, transaction validation)?

**For Crypto/Signature issues**:
1. Which signing algorithm is actually used in production? (ECDSA vs SM2)
2. Is the vulnerable function called in the production code path?
3. Are there length/format checks before the vulnerable call?
4. What Bouncy Castle version is used? Has it been patched upstream?

**For Serialization/Parsing issues**:
1. What input reaches the parser? (user-controlled vs internal)
2. Are there size limits on the input?
3. Does the parser run in a sandboxed/timeout context?
4. Can malformed input cause unbounded resource consumption?

---

## Confluence Publishing Details

| Field | Value |
|-------|-------|
| Cloud ID | `troneco.atlassian.net` |
| Space ID | `751403098` |
| Parent Page ID | `2071822585` |
| Parent Page Title | `java-tron BugBounty 分析` |
| Content Format | `markdown` |

---

## Reference

### HackerOne MCP Servers
- [Official GraphQL MCP](https://github.com/Hacker0x01/hackerone-graphql-mcp-server) — Docker-based, uses HackerOne GraphQL API
- [Community REST API MCP](https://github.com/Sicks3c/hackerone-mcp-server) — Node.js, 16 tools including `get_report`

### java-tron Repository
- GitHub: https://github.com/tronprotocol/java-tron
- Branch for analysis: `develop`

### Existing Analysis Examples
- [getBase64FromByteString() 越界问题](https://troneco.atlassian.net/wiki/spaces/javatronx/pages/1812725826) — Caller-by-caller analysis showing exception handling mitigates DoS
- [SM2 ASN1InputStream序列化问题](https://troneco.atlassian.net/wiki/spaces/javatronx/pages/1793097730) — Dead code path + already-patched dependency analysis
