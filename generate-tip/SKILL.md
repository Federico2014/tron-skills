---
name: generate-tip
description: Generate TRON Improvement Proposals (TIPs) in the official format. Use when user wants to create, draft, or write a TIP for the TRON protocol, including core protocol specifications, client APIs, or contract standards.
---

# Generate TIP

Generate TRON Improvement Proposals (TIPs) that follow the official format from [tronprotocol/tips](https://github.com/tronprotocol/tips).

## TIP Overview

TIPs describe standards for the TRON platform, including:
- Core protocol specifications
- Client APIs
- Contract standards (e.g., TRC-20, TRC-721)

## TIP Header Format

```markdown
tip: <to be assigned>
title: <TIP title> (44 characters or less)
author: <name(s) and/or username(s)>
discussions-to: <URL to GitHub issue or forum>
status: <Draft | Last Call | Accepted | Final | Deferred>
type: <Standards Track | Informational>
category: <Core | Networking | Interface | TRC | VM> (required for Standards Track)
created: <yyyy-mm-dd>
requires: <TIP number(s)> (optional)
replaces: <TIP number(s)> (optional)
```

### Author Format Examples

```
FirstName LastName (@GitHubUsername)
FirstName LastName <foo@bar.com>
FirstName (@GitHubUsername)
GitHubUsername (@GitHubUsername)
FirstName LastName (@GitHubUsername) and GitHubUsername (@GitHubUsername)
```

## TIP Types

### Standards Track
Affects most or all TRON implementations. Categories:

| Category | Description | Examples |
|----------|-------------|----------|
| Core | Consensus fork changes, core dev matters | Block validity, transaction rules |
| Networking | Network protocol improvements | P2P messages, discovery |
| Interface | Client API/RPC specifications | Wallet APIs, GRPC interfaces |
| TRC | Application-level standards | TRC-20, TRC-721 tokens |
| VM | TRON Virtual Machine changes | New opcodes, energy costs |

### Informational
Design issues, guidelines, or information that doesn't propose new features.

## TIP Statuses

| Status | Description |
|--------|-------------|
| Draft | Undergoing rapid iteration and changes |
| Last Call | Ready for wide audience review |
| Accepted | Core TIP approved by Core Devs for hard fork |
| Final | Approved and implemented (or non-Core approved) |
| Active | Never meant to be completed |
| Abandoned | No longer pursued |
| Rejected | Broken or rejected by Core Devs |
| Superseded | Replaced by newer TIP |
| Deferred | Not accepted now, may be in future |

## Required Sections

1. **Simple Summary** - Simplified explanation (layman's terms)
2. **Abstract** - ~200 word technical description
3. **Motivation** - Why existing protocol is inadequate
4. **Specification** - Syntax and semantics of new feature
5. **Rationale** - Design decisions, alternatives, community consensus
6. **Implementation** - Implementation details (required for Final status)

## Optional Sections

- **Backwards Compatibility** - Required if introducing breaking changes
- **Test Cases** - Mandatory for consensus changes

## Workflow

### Step 1: Collect Header Information

Ask user for:
1. **Title** - Must be ≤44 characters
2. **Author** - Name/email/GitHub username combinations
3. **Type** - Standards Track or Informational
4. **Category** - Core/Networking/Interface/TRC/VM (if Standards Track)
5. **Discussions-to URL** - GitHub issue or forum URL
6. **Status** - Default: Draft for new TIPs
7. **Requires/Replaces** - Optional TIP references

### Step 2: Collect Section Content

Guide user through each required section:
1. Simple Summary - What does this TIP do in simple terms?
2. Abstract - Technical summary (~200 words)
3. Motivation - What problem does this solve? Why is current protocol inadequate?
4. Specification - Detailed technical spec with syntax/semantics
5. Rationale - Why this design? What alternatives were considered?
6. Implementation - Implementation status or plans

Then ask about optional sections:
- Backwards Compatibility - Any breaking changes?
- Test Cases - Test cases for implementation?

### Step 3: Generate and Validate

Generate complete TIP with:
- Properly formatted header
- All required sections filled
- Optional sections if provided
- CC0 copyright notice

Validation:
- Title ≤44 characters
- All required fields present
- Date in ISO 8601 format (yyyy-mm-dd)

### Step 4: Output

Present TIP for review, offer to save to file.

Default filename: `tip-{title-slug}.md`

## Generated Format

```markdown
---
tip: <to be assigned>
title: <title>
author: <author>
discussions-to: <url>
status: Draft
type: <type>
category: <category>
created: <yyyy-mm-dd>
requires: <optional>
replaces: <optional>
---

## Simple Summary

<content>

## Abstract

<content>

## Motivation

<content>

## Specification

<content>

## Rationale

<content>

## Backwards Compatibility

<content or omit if not applicable>

## Test Cases

<content or omit if not applicable>

## Implementation

<content>

## Copyright

Copyright and related rights waived via [CC0](LICENSE.md).
```

## Usage Examples

**Trigger phrases:**
- "Create a TIP for..."
- "Help me write a Tron Improvement Proposal"
- "Generate a TIP for a new TRC standard"
- "I want to submit a TIP to TRON"

**Example requests:**
- "Create a TIP for a new token standard similar to ERC-1155"
- "Help me draft a TIP to add a new opcode to TVM"
- "Generate a TIP template for a networking protocol improvement"
