# Security Posture

Current security state of the IMS Azure environment, the gaps that are known and accepted, and the operational rules that keep it that way. Configuration detail lives in the individual documents — this file is the honest summary.

---

## Controls in Place

| Control | State | Detail |
|---------|-------|--------|
| MFA — all users | Enforced | Microsoft-managed policy On; user-controlled duplicate CA-001 in Report-only |
| Legacy authentication | Blocked | Microsoft-managed policy On + CA-002 On (defence in depth) |
| Compliant device required | Report-only | CA-003 — switches On in Phase 4 after Intune enrollment |
| Break-glass account | Active | Excluded from all CA policies; Severity 0 alert on any sign-in |
| SSPR | Enabled (all users) | 1 method, security questions disabled, 180-day re-confirmation |
| MDM auto-enrollment | All users | Devices auto-enroll in Intune at Entra join |
| Personally-owned devices | Blocked | Corporate registration (Autopilot) required to enroll |
| Sign-in / audit logging | Streaming | AuditLogs + SignInLogs → `law-ims-security` |

Full configuration detail: [conditional-access-policies.md](./entra-id/conditional-access-policies.md) · [emergency-access-account.md](./entra-id/emergency-access-account.md) · [sspr.md](./entra-id/sspr.md) · [enrollment-configuration.md](./intune/enrollment-configuration.md)

---

## Known Gaps and Accepted Risks

- **Sign-in risk policies (CA-004)** require Entra ID P2, which is not included in Business Premium. Deferred until a licence upgrade; documented as a planned policy in the Conditional Access doc.
- **Break-glass MFA** uses Microsoft Authenticator as a lab workaround. The production answer is a FIDO2 hardware security key stored physically alongside the printed password — phone-independent and isolated from normal infrastructure.
- **PIM is unavailable** on Business Premium, so the break-glass Global Administrator role is permanent by necessity rather than just-in-time.
- **CA-001 and CA-003 are Report-only.** MFA enforcement currently depends on the Microsoft-managed policies; CA-001 switches On after those are reviewed, and CA-003 switches On after Phase 4 device enrollment. Until then, device compliance is not enforced.

---

## Operational Rules

- Break-glass account: never used day-to-day, password stored offline only (printed copy in a physically secured location), sign-in tested every 90 days, password rotated immediately after any use. Full rules in [emergency-access-account.md](./entra-id/emergency-access-account.md).
- New Conditional Access policies start in Report-only mode. The only exception so far: CA-002, where an existing Microsoft-managed block made immediate enforcement zero-risk.
- Policy targeting uses the three `grp-ims-*` security groups ([users-and-groups.md](./entra-id/users-and-groups.md)), not individual users.
- Every configuration change is screenshotted and documented before moving on.

---

## Data Handling in This Repository

No credentials, tenant identifiers, or real staff names are committed anywhere in this repo. Placeholders (e.g. `[EMERGENCY-ACCOUNT-UPN]`, `staff01`) mark values that exist only in the tenant. When this project moves to production, real accounts exist in the tenant while the repository retains placeholders.

---

## Environment Context

Relevant to why the gaps above exist:

| Item | Value |
|------|-------|
| Licence | Microsoft 365 Business Premium (Entra ID P1 — no P2 features) |
| Tenant type | Cloud-only — no on-premises AD, no Entra Connect |
| Fleet | 11 × Windows 11 Pro (target 35) |
| MDM authority | Microsoft Intune |

---

*Last updated: July 2026*
