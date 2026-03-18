---
name: java-tron-github
description: Generate PR titles, Issue templates, and commit messages for tronprotocol/java-tron
type: skill
---

# TRON GitHub Skill

Helps you generate properly formatted commit messages, PR titles, and Issue templates for the tronprotocol/java-tron repository.

## Usage

```bash
/tron-github
```

## Commit Message Format

### Format

```
type(scope): subject
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

### Generating Commits

1. **Stage your changes**:
   ```bash
   git status                    # Review changed files
   git add path/to/file.java     # Stage specific files
   ```

2. **Determine the type**:
   - Is it a new feature? → `feat`
   - Is it fixing a bug? → `fix`
   - Is it refactoring existing code? → `refactor`
   - Is it documentation only? → `docs`
   - etc.

3. **Determine the scope**:
   - Which module did you change? (e.g., `framework`, `vm`)
   - Which feature area? (e.g., `api`, `config`)

4. **Write the subject**:
   - Start with lowercase verb (e.g., `add`, `fix`, `remove`, `update`)
   - Keep under 50 characters
   - No trailing period

5. **Create the commit**:
   ```bash
   git commit -m "type(scope): subject"
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

#### 1. Bug Report (`[BUG]`)

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

## Additional Context

**Related Issues**

**Possible Solution**
```

#### 2. Feature Request (`[FEATURE]`)

```markdown
## Background

## Problem Statement

## Rationale

**Why should this feature exist?**

**What are the use cases?**

1.
2.
3.

**Who would benefit from this feature?**

## Proposed Solution

### Specification

**API Changes**

**Configuration Changes**

**Protocol Changes**

## Testing Strategy

**Test Scenarios**

1.
2.
3.

**Performance Considerations**

## Scope of Impact

- [ ] Core protocol
- [ ] API/RPC
- [ ] Database
- [ ] Network layer
- [ ] Smart contracts
- [ ] Documentation
- [ ] Other:

**Breaking Changes**

**Backward Compatibility**

## Implementation

**Do you have ideas regarding the implementation?**

**Are you willing to implement this feature?**
- [ ] Yes, I can implement this feature
- [ ] I can help with implementation
- [ ] I need help with implementation
- [ ] I'm just suggesting the idea

**Estimated Complexity**
- [ ] Low (minor changes)
- [ ] Medium (moderate changes)
- [ ] High (significant changes)
- [ ] Unknown

## Alternatives Considered

## Additional Context

**Related Issues/PRs**

**References**
```

#### 3. Question (`[QUESTION]`)

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

## Additional Information
```

## Related Links

- Repository: https://github.com/tronprotocol/java-tron
- Documentation: https://tronprotocol.github.io/documentation-en/
- Discord: https://discord.gg/cGKSsRVCGm
- Telegram: https://t.me/TronOfficialDevelopersGroupEn

## Guidelines

### Before Submitting an Issue
- Ensure you're using the latest version of java-tron
- Search for duplicate issues
- Review the official documentation

### For General Questions
Use Discord or Telegram for faster responses.

## CI Check Requirements

PRs will automatically undergo the following checks:

### 1. PR Title Check (pr-lint)

**Format**: `type(scope): description`

**Allowed types**:
`feat`, `fix`, `refactor`, `docs`, `style`, `test`, `chore`, `ci`, `perf`, `build`, `revert`

**Known scopes**:
`framework`, `chainbase`, `actuator`, `consensus`, `common`, `crypto`, `plugins`, `protocol`, `net`, `db`, `vm`, `tvm`, `api`, `jsonrpc`, `rpc`, `http`, `event`, `config`, `block`, `proposal`, `trie`, `log`, `metrics`, `test`, `docker`, `version`, `freezeV2`, `DynamicEnergy`, `stable-coin`, `reward`, `lite`, `toolkit`

**Other rules**:
- Title length: 10-72 characters
- Description must not start with a capital letter
- Must not end with a period `.`

**Examples**:
```
feat(vm): add optimized BN128 precompiled contracts
fix(chainbase): resolve deadlock in transaction processing
```

### 2. PR Description Check

- Minimum 20 characters
- Must explain what was changed and why

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

## Branch Strategy

- **develop**: Main integration branch (default PR target)
- **master**: Production releases

### Typical Workflow

```bash
# 1. Create a feature branch from develop
git checkout develop
git pull
git checkout -b feature/your-feature-name

# 2. Make changes and commit
# ... edit files ...
git add .
git commit -m "feat(vm): add new opcode implementation"

# 3. Push and create PR
git push -u origin feature/your-feature-name
# Then create PR on GitHub targeting 'develop'
```
