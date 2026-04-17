---
name: tron-contrib
description: Generate PR titles, Issue templates, and commit messages for tronprotocol/java-tron
type: skill
---

# TRON Contrib Skill

Helps you generate properly formatted commit messages, PR titles, and Issue templates for the tronprotocol/java-tron repository.

## Usage

```bash
/tron-contrib
```

## Contributing Overview

java-tron is an open-source project that welcomes contributions from anyone. There are two main ways to contribute:

1. **Small fixes**: Send a pull request (PR) with a detailed description
2. **Complex changes**: Submit an issue to the [TIP repository](https://github.com/tronprotocol/tips) detailing your motive and implementation plan

## PR Submission Rules

### Single Module Principle

- **One PR = one problem**. Do not mix unrelated bugfixes or features.
- Split by module boundary: changes touching `net`, `consensus`, `db`, `api`, `event` etc. separately. If modules are tightly coupled (e.g. interface change requires updating callers), explain the dependency in the PR description.
- **Size limit**: excluding test file changes, **modified file count must not exceed 10**. If a PR's diff spans 2+ independent modules and exceeds this limit, split it.
  - Exceptions: bulk code relocation, or new feature with a pre-approved design doc.

### Pre-submit Checklist

Before opening a PR, verify all of the following:

| Item | Rule |
|------|------|
| Code style | Conforms to [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html), passes Checkstyle |
| No debug artifacts | No `System.out.println`, no leftover temporary comments, no unresolved `TODO` |
| Numeric safety | 1. No `java.lang.Math` or `Maths` helper class — use `StrictMathWrapper` directly. 2. No floating-point in consensus code — use `BigDecimal` / `BigInteger` / `xxxExact`. 3. Type casts use range-checking methods (`xxxExact`). |
| Log levels | Avoid `INFO`/`WARN` on high-frequency code paths |
| DB change | Include backward compatibility explanation |
| Consensus change | Document hard fork impact and which node versions need upgrading |
| Config change | Sync updates to `config.conf` and all related config files |
| Dependency upgrade | State upgrade reason and compatibility verification result |
| Comments | Complex logic must have inline comments |
| Naming | No ambiguous, misleading, or hard-to-read variable / method / class names |

### Keeping the PR Updated

- Respond to review comments **within 3 business days** (acknowledge, push back, or explain progress).
- After code changes: **Resolve conversation** on the relevant comment or leave a reply, and update the PR description with a change summary.
- If blocked by leave or a dependency, arrange a substitute reviewer.
- PRs with **no update for 3+ business days without explanation** will be pinged by the maintainer and may be marked stale.

---

## Commit Message Format

### Format

```
type(scope): subject
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

- **subject**: Under 50 characters, lowercase verb, no trailing period
- **type**: One of the allowed types (see below)
- **scope**: Module or feature scope (see below)

### Allowed Types

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Code refactoring |
| `docs` | Documentation changes |
| `style` | Code style changes (formatting, etc.) |
| `test` | Test additions or modifications |
| `chore` | Maintenance tasks |
| `ci` | CI/CD configuration changes |
| `perf` | Performance improvements |
| `build` | Build system changes |
| `revert` | Reverting previous commits |

### Known Scopes

**Module scopes**: `framework`, `chainbase`, `actuator`, `consensus`, `common`, `crypto`, `plugins`, `protocol`

**Feature scopes**: `net`, `db`, `api`, `config`, `vm`, `tvm`, `event`, `tool`, `log`, `jsonrpc`, `rpc`, `http`, `block`, `proposal`, `trie`, `metrics`, `docker`, `version`, `freezeV2`, `DynamicEnergy`, `stable-coin`, `reward`, `lite`, `toolkit`

### Commit Examples

```bash
# Good commits
git commit -m "feat(vm): add optimized BN128 precompiled contracts"
git commit -m "fix(chainbase): resolve deadlock in transaction processing"
git commit -m "refactor(config): extract CLIParameter and restructure Args init flow"
git commit -m "docs(readme): update build instructions"
git commit -m "ci: add multi-platform build tests"

# Bad commits (avoid these)
git commit -m "feat: Add New Feature"     # 'Add' should be lowercase
git commit -m "fix: fixed a bug."         # no trailing period
git commit -m "updated stuff"             # missing type and scope
git commit -m "feat(scope): this subject is way too long and exceeds the fifty character limit"
```

### Commit Message Template

```
<commit type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

**Example:**
```
feat(block): optimize the block-producing logic

1. increase the priority that block producing thread acquires synchronization lock
2. add the interruption exception handling in block-producing thread

Closes #1234
```

**Footer for issues:**
- Single issue: `Closes #1234`
- Multiple issues: `Closes #123, #245, #992`

## Branch Strategy

### Key Branches

| Branch | Description |
|--------|-------------|
| `develop` | Main integration branch. Default PR target. Cannot be directly pushed to. |
| `master` | Production releases. Only `release_*` and `hotfix/*` branches merge here. |
| `release_*` | Release branches pulled from `develop`. Permanently kept. |
| `feature/*` | Feature branches for important features. Pulled from `develop`. |
| `hotfix/*` | Bug fix branches pulled from `master`. Used for post-release fixes. |

### Branch Naming Conventions

| Branch Type | Naming Pattern | Examples |
|-------------|---------------|----------|
| Master | `master` | master |
| Development | `develop` | develop |
| Release | Version number | `Odyssey-v3.1.3`, `3.1.3` |
| Hotfix | `hotfix/<bug-description>` | `hotfix/typo`, `hotfix/null_point_exception` |
| Feature | `feature/<feature-description>` | `feature/new_resource_model` |

### Typical Workflow

```bash
# 1. Fork the repository
# Go to https://github.com/tronprotocol/java-tron and click "Fork"

# 2. Clone your fork
git clone https://github.com/yourname/java-tron.git
cd java-tron

# 3. Add upstream remote
git remote add upstream https://github.com/tronprotocol/java-tron.git

# 4. Sync with upstream before developing
git fetch upstream
git checkout develop
git merge upstream/develop --no-ff

# 5. Create feature branch
git checkout -b feature/your-feature-name develop

# 6. Make changes and commit
# ... edit files ...
git add .
git commit -m "feat(vm): add new opcode implementation"

# 7. Push to your fork
git push origin feature/your-feature-name

# 8. Create PR on GitHub
# Go to tronprotocol/java-tron and create PR from your branch
```

## Merge Strategy

Core principle: **feature into trunk → Squash; branch sync → Rebase; merge commit → disabled by default**.

| Method | When to use | Branch direction |
|--------|-------------|-----------------|
| **Squash and Merge** | Feature PRs | `feature → develop / release_xxx / master` — squashes iteration commits into one, keeps trunk history clean |
| **Rebase and Merge** | Branch sync | `release_xxx → master`, `master → develop` — preserves individual feature commits, avoids meaningless merge nodes |
| **Create a merge commit** | Almost never | Only in special cases where an explicit merge record is needed |

---

## Templates

### Pull Request Template

```markdown
**What does this PR do?**

**Why are these changes required?**

**This PR has been tested by:**
- Unit Tests
- Manual Testing

**Checklist**
- [ ] Single module — this PR only touches one problem / one module boundary
- [ ] No `java.lang.Math` or `Maths` — uses `StrictMathWrapper` instead
- [ ] No floating-point in consensus code
- [ ] No `System.out.println` or leftover TODOs
- [ ] Complex logic has comments
- [ ] If consensus change: hard fork impact explained and node version range noted
- [ ] If DB change: backward compatibility explained
- [ ] If config change: `config.conf` and related files updated

<!-- AI assistance disclosure (encouraged, not required) -->
<!-- > AI 辅助说明：本 PR 部分代码借助 AI 工具生成，已逐行审查，提交者承担完全责任。-->

**Follow up**

**Extra details**
```

### Issue Templates

#### 1. Bug Report (`[Bug]`)

```markdown
## Bug Description

## Environment

**Network**

**Software Versions**
```
OS:
JVM:
Git Commit:
Version:
Code:
```

**Configuration**

## Expected Behavior

## Actual Behavior

## Frequency
- [ ] Always (100%)
- [ ] Frequently (>50%)
- [ ] Sometimes (10-50%)
- [ ] Rarely (<10%)

## Steps to Reproduce

1.
2.
3.

## Logs and Error Messages

```
```

## Additional Context (Optional)

**Screenshots**
<!-- If applicable, add screenshots to help explain the problem -->

**Related Issues**
<!-- Link to any related issues or pull requests -->

**Possible Solution**
<!-- If you have suggestions on how to fix the bug, describe them here -->
```

#### 2. Feature Request (`[Feature]`)

```markdown
# Summary
<!-- Provide a concise description of the problem and the proposed solution.-->

# Problem
### Motivation
<!-- Describe the context or motivation if necessary. -->

### Current State
<!-- Describe the current behavior of the system. -->

### Limitations or Risks
<!-- Explain the limitations, risks, or inefficiencies. -->

# Proposed Solution

### Proposed Design
<!-- Describe the proposed solution at a high level. Implementation details are optional but encouraged if relevant.-->

### Key Changes
<!-- List the main areas affected by this proposal, such as Module(s), Configuration and API. -->

# Impact
<!-- Assess the expected impact of this change, such as Security, Stability, Performance, Developer Experience   -->

# Compatibility
<!--
*   Breaking Change: Yes / No
*   Default Behavior Change:
*   Migration Required:
Provide details if any of the above is applicable.
-->

# References (Optional)
<!-- TIPs, papers, related issues, prior art -->

# Additional Notes
- Do you have ideas regarding implementation? Yes / No
- Are you willing to implement this feature? Yes / No
```

#### 3. Question (`[Question]`)

```markdown
## Question

## Context

**What are you trying to achieve?**

**What have you tried so far?**

**Relevant documentation or code**

## Environment

- Network:
- java-tron version:
- Operating System:
- Java version:

## Additional Information (Optional)
```

## Code Review Guidelines

### Terminology

- **Author**: Entity who wrote the diff and submitted it to GitHub
- **Team**: People with commit rights on the java-tron repository
- **Reviewer**: Person assigned to review the diff (must be a team member)
- **Code Owner**: Person responsible for the subsystem being modified

### The Process

1. PR is evaluated for worthiness (decision lies with code owner)
2. Reviewers check style and functionality, providing comments via GitHub review system
3. Author addresses feedback within 3 business days
4. Approved PRs can be merged once merge requirements are met

### Merge Requirements

- At least **2 reviewer Approvals** (both must be core maintainers of the affected module)
- Changes to consensus, storage, or network core paths: **3 Approvals required**
- All CI checks (build, unit tests, Checkstyle) must pass
- All `[MUST]` comments must be resolved
- Submitter may not Approve their own PR (including via alt accounts)

### Code Style Requirements

- Must conform to [Google Code Style](https://google.github.io/styleguide/javaguide.html)
- Must pass Sonar scanner test
- Must pass Travis CI continuous integration scanner
- Code must be pulled from `develop` branch
- Commit message should start with a verb (lowercase first letter)
- Commit message should be under 50 characters

## CI Check Requirements

PRs will automatically undergo the following checks:

### 1. PR Title Check (pr-lint)

**Format**: `type(scope): description`

| Rule | Requirement |
|------|-------------|
| Format | `type(scope): description` |
| Length | 10-72 characters |
| Types | `feat`, `fix`, `refactor`, `docs`, `style`, `test`, `chore`, `ci`, `perf`, `build`, `revert` |
| Description | Must not start with capital letter, must not end with period |

**Known scopes**:
`framework`, `chainbase`, `actuator`, `consensus`, `common`, `crypto`, `plugins`, `protocol`, `net`, `db`, `vm`, `tvm`, `api`, `jsonrpc`, `rpc`, `http`, `event`, `config`, `block`, `proposal`, `trie`, `log`, `metrics`, `test`, `docker`, `version`, `freezeV2`, `DynamicEnergy`, `stable-coin`, `reward`, `lite`, `toolkit`

### 2. PR Description Check

- Minimum 20 characters
- Must explain **what** was changed and **why**

### 3. Checkstyle Check

- Enforced on `framework` and `plugins` modules
- Config: `framework/config/checkstyle/checkStyleAll.xml`
- Max line length: 100 characters
- Indentation: 2 spaces (4 for line wraps)
- No star imports

### 4. Math Usage Check (math-check)

- Forbidden: `java.lang.Math`
- Use: `org.tron.common.math.StrictMathWrapper`
- Exceptions: `StrictMathWrapper.java` and `MathWrapper.java`

### 5. Build Check

- Multi-platform: Ubuntu/macOS (x86_64/aarch64)
- JDK 8 and JDK 17

## Local Verification Commands

```bash
# Check current status and recent changes
git status
git log -3 --oneline

# Stage changes
git add path/to/file.java

# Build project (skip tests)
./gradlew clean build -x test

# Run checkstyle
./gradlew :framework:checkstyleMain :framework:checkstyleTest :plugins:checkstyleMain

# View diff before committing
git diff
```

## Special Situations

| Situation | How to Deal |
|-----------|-------------|
| Author doesn't follow up | Ping after 3 business days. If no response, maintainer marks stale; closed after 7 more days. |
| Author mixes refactoring with bug fix | Ask author to separate into independent PRs or commits. |
| Author keeps rejecting feedback | Reviewers have authority to reject. Ask team for second opinion if unsure. Close PR if no consensus. |

### Hotfix

- Mainnet emergency fixes may use an **accelerated path**: 1 core maintainer Approve is sufficient to merge.
- Must be followed up within 7 days with complete tests and documentation.

### AI-Assisted Code

Allowed: AI tools for code completion, logic suggestions, test generation.

Prohibited: submitting AI output without fully understanding it. Before submitting, you must be able to:
- Explain every line of logic and its meaning in the java-tron context
- Justify why this implementation is better than alternatives
- Answer any follow-up question from reviewers in depth

Prohibited in consensus / storage / crypto modules: directly using AI-generated core logic — errors in these paths can cause irreversible on-chain state corruption.

AI identifiers must not appear in commit author fields.

Disclosure is encouraged (not required). Suggested format for PR description:
```
> AI 辅助说明：本 PR 部分代码（测试用例/注释/模板代码）借助 AI 工具生成，
> 已逐行审查并验证，提交者对所有内容承担完全责任。
```

## Conduct

While contributing, please be respectful and constructive. Participation in the project should be a positive experience for everyone.

**Acceptable behavior:**
- Using welcoming and inclusive language
- Being respectful of differing viewpoints and experiences
- Gracefully accepting constructive criticism
- Focusing on what is best for the community
- Showing empathy towards other community members

**Unacceptable behavior:**
- Sexualized language or imagery
- Trolling, insulting/derogatory comments, personal or political attacks
- Public or private harassment
- Publishing others' private information without permission
- Other inappropriate conduct in a professional setting

## Related Links

- Repository: https://github.com/tronprotocol/java-tron
- Documentation: https://tronprotocol.github.io/documentation-en/
- TIP Repository: https://github.com/tronprotocol/tips
- Discord: https://discord.gg/cGKSsRVCGm
- Telegram: https://t.me/TronOfficialDevelopersGroupEn

## Guidelines

### Before Submitting an Issue
- Ensure you're using the latest version of java-tron
- Search for duplicate issues
- Review the official documentation

### For General Questions
Use Discord or Telegram for faster responses.
