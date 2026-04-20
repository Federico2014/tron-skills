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

- One PR solves one problem. Never mix unrelated bugfixes or features into the same PR.
- Split by module boundary: changes spanning net, consensus, db, API, event, or other independent sub-modules must be submitted separately.
- **File count limit**: non-test file changes must not exceed **10 files**. Exceptions:
  - Wholesale code relocation
  - New features that passed a prior design review
- When strong cross-module dependencies exist (e.g., an interface change that requires updating callers), state the relationship explicitly in the PR description.

### Pre-submit Checklist

| Item | Requirement |
|------|-------------|
| Code style | Conforms to Google Java Style Guide; passes Checkstyle |
| No debug code | No `System.out.println`, temporary comments, or unresolved TODOs |
| Numeric safety | ① Do not use `java.lang.Math` or the legacy `Maths` helper — use `StrictMathWrapper` instead; ② No floating-point in consensus code — use `BigDecimal`; integer arithmetic must use `BigInteger` or `StrictMathWrapper.xxxExact`; ③ Narrowing casts must use `xxxExact` range-checking methods |
| Logging levels | Avoid redundant INFO/WARN logs on hot paths |
| Database changes | Must include a compatibility statement |
| Consensus changes | Must describe hard-fork impact and the node version range that needs upgrading |
| Config changes | Must sync `config.conf` and related configuration files |
| Dependency upgrades | State the upgrade reason and compatibility verification; attach a security scan report (e.g., OWASP dependency-check output) and assess impact on node startup time and memory |
| Comments | Complex logic must have explanatory comments |
| Naming | No ambiguous, near-duplicate, or unreadable variable / method / class names |

### Submitter Response SLA

- Respond to review feedback within **3 business days** (acknowledge, push back, or explain progress).
- After fixing code, Resolve the conversation or reply with a summary, and update the PR description with a change summary.
- If unable to respond due to objective reasons (vacation, blocking dependency), arrange a substitute reviewer.
- PRs with no update for more than 3 business days without explanation may be marked `stale`.

### AI-Assisted Development Disclosure

When using AI tools (Copilot, Cursor, Claude, etc.):
- **Allowed**: code completion, logic suggestions, test generation
- **Prohibited**: directly using AI-generated core logic in consensus, storage, or cryptography modules
- **Prohibited**: commit author field must not include AI identifiers
- The submitter must be able to explain every line of code and answer any follow-up questions from reviewers in depth

Encouraged (not mandatory) disclosure in the PR description:
```
> AI assistance note: parts of this PR (test cases / comments / boilerplate) were
> generated with AI tooling, reviewed line-by-line, and verified by the submitter,
> who takes full responsibility for all content.
```

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

### Merge Strategy

| Strategy | Use case | Branch direction |
|----------|---------|-----------------|
| **Squash and Merge** | Feature development PR | feature → develop / release_xxx / master |
| **Rebase and Merge** | Branch sync | release_xxx → master, master → develop |
| Create a merge commit | Rarely used | — |

- Feature into main branch → Squash (one PR = one commit; keeps history clean and easy to revert)
- Branch-to-branch sync → Rebase (preserves per-feature commit granularity; avoids meaningless merge nodes)

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

## Templates

### Pull Request Template

```markdown
**What does this PR do?**

**Why are these changes required?**

**This PR has been tested by:**
- Unit Tests
- Manual Testing

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
3. Author addresses feedback
4. Approved PRs can be merged by any code owner

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
| Author doesn't follow up | Ping after a few days. If no response, close PR or complete work yourself. |
| Author mixes refactoring with bug fix | Ask author to separate into independent PRs or commits. |
| Author keeps rejecting feedback | Reviewers have authority to reject. Ask team for second opinion if unsure. Close PR if no consensus. |

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
