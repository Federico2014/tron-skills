# TRON Skills

A collection of Skills for TRON blockchain development.

## Available Skills

| Skill | Description |
|-------|-------------|
| [tron-contrib](#tron-contrib) | Generate PR titles, Issue templates, and commit messages for tronprotocol/java-tron |
| [tron-tip](#tron-tip) | Generate TRON Improvement Proposals (TIPs) in the official format |

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

## Related Resources

- [java-tron Repository](https://github.com/tronprotocol/java-tron)
- [Official Documentation](https://tronprotocol.github.io/documentation-en/)
- [Discord Community](https://discord.gg/cGKSsRVCGm)
- [Telegram Developer Group](https://t.me/TronOfficialDevelopersGroupEn)
