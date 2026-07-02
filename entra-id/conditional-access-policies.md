# Conditional Access Policies

## Overview

Conditional Access is the policy engine in Microsoft Entra ID that controls who can access what, from where, and under what conditions. It sits between the user and the resource — evaluating signals like user identity, device compliance, location, and risk level before granting access.

The break-glass emergency access account is excluded from every policy in this document. See [emergency-access-account.md](./emergency-access-account.md) for the reasoning.

---

## Microsoft-Managed Policies (Auto-Created)

When Microsoft 365 Business Premium was provisioned, Microsoft automatically created four Conditional Access policies. These are active by default.

### 1. Multifactor authentication for all users

| Setting | Value |
|---------|-------|
| Created by | Microsoft |
| State | On |
| Included identities | All users |
| Excluded identities | Break-glass account |
| Cloud apps | All apps |
| Requirement | MFA |

Requires MFA for every user across all Microsoft cloud apps. Break-glass is excluded.

### 2. Multifactor authentication for admins

| Setting | Value |
|---------|-------|
| Created by | Microsoft |
| State | On |
| Included identities | All users with admin roles |
| Excluded identities | Break-glass account |
| Cloud apps | All apps |
| Requirement | MFA |

Adds a specific MFA requirement targeting accounts assigned admin roles. The break-glass account holds Global Administrator but is excluded — consistent with emergency access design.

### 3. Multifactor authentication for Azure Management

| Setting | Value |
|---------|-------|
| Created by | Microsoft |
| State | On |
| Included identities | All users |
| Excluded identities | Break-glass account |
| Cloud apps | Azure portal, Entra admin center, Intune admin center |
| Requirement | MFA |

Specifically targets sign-ins to Azure management portals. Break-glass is excluded.

### 4. Block legacy authentication

| Setting | Value |
|---------|-------|
| Created by | Microsoft |
| State | On |
| Included identities | All users |
| Client apps | Exchange ActiveSync, Other clients (legacy auth protocols) |
| Grant | Block access |

Blocks all legacy authentication protocols — SMTP, POP3, IMAP, basic auth. These protocols cannot enforce MFA, making them the most common vector for Microsoft 365 account takeovers. This policy has no exclusions because legacy auth has no legitimate use case in a managed environment.

---

## User-Created Policies

The following policies will be built as part of Phase 2. This section will be updated as each policy is created and tested.

### CA-001 — Require MFA for All Users
- **Status:** Planned
- Users: All users / Exclude: Break-glass
- Apps: All cloud apps
- Grant: Require MFA
- Deployment: Report-only first, then Enabled

### CA-002 — Block Legacy Authentication (User Policy)
- **Status:** Planned
- Users: All users
- Client apps: Exchange ActiveSync + Other clients
- Grant: Block access
- Deployment: Enabled immediately

### CA-003 — Require Compliant Device
- **Status:** Planned
- Users: All users / Exclude: Break-glass
- Apps: Microsoft 365 (Exchange, SharePoint, Teams)
- Grant: Require device marked as compliant
- Deployment: Report-only until all devices are enrolled and compliant in Intune, then Enabled

### CA-004 — Sign-In Risk Block
- **Status:** Requires Entra ID P2 — not available on Business Premium
- This policy will be documented here when the licence is upgraded to P2
- Planned: Block access on high sign-in risk

---

## Notes

**Security Defaults vs Conditional Access:** These cannot run at the same time. When Microsoft-managed CA policies are active, Security Defaults are automatically disabled. This tenant runs on Conditional Access.

**Policy order:** Conditional Access policies do not have a priority order — all matching policies are evaluated and the most restrictive result wins. An explicit Block always overrides a Grant.

**Testing approach:** Each new policy is created in Report-only mode first. This lets you see what would have been blocked before enforcement begins, without locking anyone out.

---

*Last updated: July 2026*
