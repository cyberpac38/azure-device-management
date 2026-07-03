# Conditional Access Policies

## Overview

Conditional Access is the policy engine in Microsoft Entra ID that controls who can access what, from where, and under what conditions. Every sign-in request is evaluated against all matching policies simultaneously — the most restrictive result wins. An explicit Block always overrides a Grant.

The break-glass emergency access account is excluded from every policy in this document. See [emergency-access-account.md](./emergency-access-account.md) for the reasoning.

---

## Microsoft-Managed Policies

When Microsoft 365 Business Premium was provisioned, Microsoft automatically created four Conditional Access policies, all active by default.

### 1. Multifactor authentication for all users

Requires MFA for every user across all Microsoft cloud apps. Break-glass is excluded (1 user excluded, 0 groups, 0 roles).

*Verification Log — Policy detail showing break-glass exclusion and State: On:*

![MFA for all users - break-glass excluded](./screenshots/01-ca-policy-mfa-breakglass-excluded.png)

> **Design Decision — Microsoft-Managed vs User-Created:** Microsoft-managed policies cannot be fully customised. A user-created equivalent (CA-001) is maintained in Report-only mode alongside this policy. This gives the organisation a path to full user control over enforcement without disabling Microsoft protection in the meantime.

### 2. Multifactor authentication for admins

Targets accounts assigned admin roles. Break-glass holds Global Administrator but is excluded — consistent with emergency access design.

### 3. Multifactor authentication for Azure Management

Targets sign-ins to the Azure portal, Entra admin center, and Intune admin center specifically. Break-glass is excluded.

### 4. Block legacy authentication

Blocks all legacy authentication protocols — Exchange ActiveSync, basic auth, SMTP, POP3, IMAP. These protocols cannot enforce MFA and are the most common vector for Microsoft 365 account compromise. No exclusions applied — there is no legitimate use case for legacy auth in a managed environment.

**Real-world discovery:** Security Defaults were already disabled on this tenant because Microsoft-managed CA policies were active. Security Defaults and Conditional Access cannot run simultaneously — no manual action was required.

---

## User-Created Policies

The following policies are built alongside the Microsoft-managed policies to give the organisation full control over enforcement. Each new policy starts in Report-only mode.

*Verification Log — Complete policy list (Microsoft-managed + user-created):*

![CA policies complete list](./screenshots/10-ca-policies-list.png)

> *(Screenshot to be updated after CA-001, CA-002, CA-003 are confirmed created)*

### CA-001 — Require MFA for All Users

| Setting | Value |
|---------|-------|
| Status | Report-only |
| Users | All users |
| Exclude | Break-glass account |
| Apps | All cloud apps |
| Grant | Require multifactor authentication |

Mirrors the Microsoft-managed MFA policy. Running in Report-only to monitor impact before enforcement begins.

### CA-002 — Block Legacy Authentication

| Setting | Value |
|---------|-------|
| Status | Enabled |
| Users | All users |
| Client apps | Exchange ActiveSync, Other clients |
| Grant | Block access |

Mirrors the Microsoft-managed block policy. No exclusions — enabled immediately.

### CA-003 — Require Compliant Device

| Setting | Value |
|---------|-------|
| Status | Report-only |
| Users | All users |
| Exclude | Break-glass account |
| Apps | Office 365 (Exchange, SharePoint, Teams) |
| Grant | Require device marked as compliant |

> **Design Decision — Report-Only Until Intune Enrollment Complete:** Enabling this policy before all 11 IMS devices are enrolled and marked compliant in Intune would immediately block all staff from accessing Microsoft 365. Report-only mode allows the compliance posture to be monitored and corrected before enforcement begins. The policy switches to Enabled only after Phase 4 device enrollment is complete.

### CA-004 — Sign-In Risk Block

| Setting | Value |
|---------|-------|
| Status | Not available |
| Reason | Requires Entra ID P2 — this tenant runs Business Premium (P1 only) |

Will be implemented when the licence is upgraded to P2.

---

## Notes

**Testing approach:** Every new policy is created in Report-only mode first. This shows what would have been blocked without enforcing anything, allowing issues to be identified and corrected before users are impacted.

**Policy evaluation:** All matching policies are applied simultaneously. There is no priority order — a Block from one policy overrides a Grant from another.

---

*Last updated: July 2026*
