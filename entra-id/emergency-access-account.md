# Emergency Access Account (Break-Glass)

## What It Is

An emergency access account is a cloud-only Global Administrator account used only when normal admin access is broken — for example, a misconfigured Conditional Access policy that locks out all administrators, or an MFA outage affecting the primary admin account.

Without this account, a lockout scenario requires calling Microsoft support. With it, you can get back into the tenant immediately and fix the problem yourself.

## What Was Created

- **Account type:** Cloud-only (not synced from on-premises)
- **Role:** Global Administrator (permanent — not eligible via PIM)
- **License:** None (unlicensed by design)
- **MFA:** FIDO2 hardware security key (production requirement — see note below)

## Conditional Access Exclusions

The break-glass account is excluded from all Conditional Access policies. This is intentional and permanent.

If a CA policy is the reason admins are locked out, the break-glass account must not be subject to that same policy — otherwise you have no recovery path.

The account is excluded from the following policies:

| Policy | Type | Break-Glass Excluded |
|--------|------|---------------------|
| Multifactor authentication for all users | Microsoft-managed | Yes |
| Multifactor authentication for admins | Microsoft-managed | Yes |
| Multifactor authentication for Azure Management | Microsoft-managed | Yes |
| Block legacy authentication | Microsoft-managed | N/A |

As additional Conditional Access policies are created, the break-glass account will be excluded from each one.

## MFA Requirement — Important Discovery

As of October 2024, Microsoft enforces MFA at the platform level for all sign-ins to the Azure portal, Entra admin center, and Intune admin center. This applies to all accounts including emergency access accounts and cannot be bypassed by excluding the account from Conditional Access policies.

This means the traditional approach of a break-glass account with only a username and password no longer works for portal access.

**Production solution:** Register the break-glass account with a FIDO2 hardware security key (e.g. YubiKey). The physical key is stored in a secure location alongside the printed password. This satisfies MFA without depending on a phone, authenticator app, or internet connectivity — making it genuinely independent of normal infrastructure.

**Lab workaround used:** Microsoft Authenticator app registered. Replace with FIDO2 key before production deployment.

Reference: [Plan for mandatory Microsoft Entra MFA](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-mandatory-multifactor-authentication)

## Sign-In Alert

An Azure Monitor alert fires any time the break-glass account signs in successfully. The alert is configured as Severity 0 — Critical and sends an immediate email notification.

Any use of this account outside of a declared emergency should be treated as a security incident.

**Alert details:**
- Name: alert-breakglass-signin
- Severity: 0 — Critical
- Signal type: Log search (KQL against Log Analytics workspace)
- Frequency: Evaluated every 5 minutes

**KQL query used:**
```kql
SigninLogs
| where UserPrincipalName == "[EMERGENCY-ACCOUNT-UPN]"
| where ResultType == 0
```

> Replace `[EMERGENCY-ACCOUNT-UPN]` with the actual UPN of your emergency access account. Never publish the real username in a public repository.

Sign-in logs flow from Entra ID to a Log Analytics workspace via diagnostic settings (AuditLogs + SignInLogs).

## Operational Rules

- Never use this account for day-to-day administration
- Never store the password digitally — printed copy only, stored physically in a secure location
- Never add this account to any group or assign it additional roles beyond Global Administrator
- Test the account signs in successfully at least once every 90 days
- If the account is used for any reason, rotate the password immediately after and document why it was used
