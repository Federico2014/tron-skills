# TRON Skills

A collection of Skills for TRON blockchain development.

## Available Skills

| Skill | Description |
|-------|-------------|
| [tron-contrib](#tron-contrib) | Generate PR titles, Issue templates, and commit messages for tronprotocol/java-tron |
| [tron-tip](#tron-tip) | Generate TRON Improvement Proposals (TIPs) in the official format |
| [tron-pr-review](#tron-pr-review) | PR code review for java-tron with copy-paste-ready GitHub inline comments |
| [tron-issue-comment](#tron-issue-comment) | Write copy-paste-ready GitHub issue comments for tronprotocol/java-tron |
| [weekly-report](#weekly-report) | Auto-collect data from Slack & GitHub to generate Federico's weekly report |
| [bugbounty](#bugbounty) | Review HackerOne bug bounty reports against java-tron code and publish analysis to Confluence |

---

### tron-contrib

**Path**: `tron-contrib/`

**Description**: Generates properly formatted PR titles, Issue templates, and commit messages for the `tronprotocol/java-tron` repository.

**Usage**:
```bash
/tron-contrib
```

**Features**:

- **Commit Message Generation** - Follows conventional commit format `type(scope): subject`
- **PR Templates** - Complete Pull Request description framework
- **Issue Templates** - Supports Bug Reports, Feature Requests, and Questions
- **CI Check Guidelines** - Local verification commands and branch strategy

**Commit Format Examples**:
```bash
feat(vm): add optimized BN128 precompiled contracts
fix(chainbase): resolve deadlock in transaction processing
refactor(config): extract CLIParameter and restructure Args init flow
```

**Supported Commit Types**: `feat`, `fix`, `refactor`, `docs`, `style`, `test`, `chore`, `ci`, `perf`, `build`, `revert`

**Supported Scopes**: `framework`, `chainbase`, `actuator`, `consensus`, `common`, `crypto`, `plugins`, `protocol`, `vm`, `tvm`, `api`, `jsonrpc`, `db`, `net`, and more

---

### tron-tip

**Path**: `tron-tip/`

**Description**: Generate TRON Improvement Proposals (TIPs) in the official format from [tronprotocol/tips](https://github.com/tronprotocol/tips).

**Usage**:
```bash
/tron-tip
```

**Features**:

- **TIP Header Generation** - Properly formatted metadata (tip, title, author, status, type, category, created, etc.)
- **Required Sections** - Simple Summary, Abstract, Motivation, Specification, Rationale, Implementation
- **Optional Sections** - Backwards Compatibility, Test Cases
- **Validation** - Title length check (≤44 characters), required fields, date format

**TIP Types**:

| Type | Description |
|------|-------------|
| Standards Track | Affects TRON implementations (Core, Networking, Interface, TRC, VM) |
| Informational | Design issues, guidelines, or informational content |

**TIP Categories** (Standards Track only):
- `Core` - Consensus, block validity, transaction rules
- `Networking` - P2P messages, discovery protocol
- `Interface` - Client API, RPC specifications
- `TRC` - Application-level standards (TRC-20, TRC-721)
- `VM` - TVM opcodes, energy costs

**Usage Examples**:
```
"Create a TIP for a new token standard similar to ERC-1155"
"Help me draft a TIP to add a new opcode to TVM"
"Generate a TIP template for a networking protocol improvement"
```

---

### tron-pr-review

**Path**: `tron-pr-review/`

**Description**: PR code review for java-tron (tronprotocol/java-tron). Produces copy-paste-ready GitHub inline comments grouped by file with verified line numbers and severity tags.

**Usage**:
```bash
/tron-pr-review
```

**Features**:

- **Inline Comments** - Grouped by file with exact line numbers verified against source
- **Severity Tags** - `[Critical]` `[High]` `[Medium]` `[Low]` `[Nit]`
- **Review Categories** - Security, Business Logic, Behavioral Compatibility, Lifecycle & Resource Safety, Concurrency, Test Quality, Code Style
- **Verdict Summary** - `LGTM`, `LGTM with nits`, or `Changes requested (N blockers)`

**Trigger phrases**:
```
"review this PR"
"review the diff"
"/tron-pr-review"
"帮我review"
```

---

### tron-issue-comment

**Path**: `tron-issue-comment/`

**Description**: Write copy-paste-ready GitHub issue comments for tronprotocol/java-tron. Covers bug analysis, feature request feedback, needs-more-info, and close/resolve replies.

**Usage**:
```bash
/tron-issue-comment
```

**Features**:

- **Bug Analysis** - Root cause, affected component, fix direction
- **Feature Request Feedback** - Assessment, trade-offs, recommendation (Accept / Defer / Needs TIP / Won't implement)
- **Needs More Info** - Structured checklist for reproduction steps, version, logs, config
- **Close / Resolve** - Fixed, duplicate, won't-fix, or out-of-scope closers

**Trigger phrases**:
```
"write an issue comment"
"reply to this issue"
"帮我回复这个issue"
"/tron-issue-comment"
```

---

### weekly-report

**Path**: `weekly-report/`

**Description**: Auto-collect work data from Slack and GitHub, generate Federico's weekly report in Confluence table format (This Week's Work / Issues Encountered / Next Week's Plan).

**Usage**:
```bash
/weekly-report
```

**Features**:

- **Auto Time Range** - Calculates last Friday 12:00 → this Friday 12:00
- **Multi-Source Collection** - Slack daily notes, GitHub PRs/commits/reviews/issues
- **Three-Table Output** - This Week's Work, Issues Encountered, Next Week's Plan
- **Smart Carry-Forward** - Incomplete items auto-populate next week's plan
- **Confluence Format** - Output in XHTML Storage Format for direct paste

**Data Sources**:

| Source | Tool |
|--------|------|
| Slack daily-ai-assistant | Slack MCP |
| GitHub (Federico2014) | `gh` CLI |
| GitHub (tronprotocol) | `gh` CLI |
| Confluence | Atlassian MCP (pending) |

**Trigger phrases**:
```
"写周报"
"生成周报"
"weekly report"
"/weekly-report"
```

---

### bugbounty

**Path**: `bugbounty/`

**Description**: Review HackerOne bug bounty reports against the java-tron codebase. Fetches report details via HackerOne MCP, performs deep code analysis on the develop branch, assesses vulnerability validity, and publishes structured analysis to Confluence.

**Usage**:
```bash
/bugbounty
```

**Features**:

- **Report Fetching** - Retrieves HackerOne reports by number via MCP (GraphQL or REST API)
- **Code Analysis** - Traces all callers, validates PoC, checks existing mitigations
- **Verdict System** - Valid / Partially Valid / Invalid / Informational / Already Fixed
- **Confluence Publishing** - Creates child page under java-tron BugBounty with structured analysis

**Analysis Patterns**:
- DoS vulnerabilities: reachability, rate limiting, exception handling
- Crypto/Signature issues: production code path, dependency versions
- Serialization issues: input validation, resource consumption bounds

**Trigger phrases**:
```
"review bug bounty"
"analyze hackerone report"
"/bugbounty"
```

---

## Related Resources

- [java-tron Repository](https://github.com/tronprotocol/java-tron)
- [Official Documentation](https://tronprotocol.github.io/documentation-en/)
- [Discord Community](https://discord.gg/cGKSsRVCGm)
- [Telegram Developer Group](https://t.me/TronOfficialDevelopersGroupEn)
