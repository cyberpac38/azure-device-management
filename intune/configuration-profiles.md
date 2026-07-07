# Intune Configuration Profiles

## Overview

The compliance policy (`Win11-Baseline-Compliance`) defines what a healthy device looks like; these profiles are what *make* devices healthy. A device that enrolls with only the compliance policy assigned would sit noncompliant with no remediation path — the policy checks BitLocker, but nothing turns BitLocker on. Each profile below maps directly to a compliance requirement.

| Profile | Enforces | Satisfies compliance check |
|---------|----------|---------------------------|
| Win11-BitLocker-Encryption | Silent full-disk encryption, keys escrowed to Entra ID | BitLocker: Require |
| Windows Hello for Business | Passwordless sign-in (PIN + biometrics backed by TPM) | Password to unlock: Require |
| Update ring | Windows Update for Business deferral/deadline policy | (Replaces manual Ansible update scheduling) |

---

## Win11-BitLocker-Encryption

Endpoint security > Disk encryption > BitLocker · Platform: Windows

### Base settings

| Setting | Value |
|---------|-------|
| Require Device Encryption | Enabled |
| Allow Warning For Other Disk Encryption | **Disabled** — this is what makes encryption silent |
| Allow Standard User Encryption | Enabled |

*Verification Log — base settings:*

![BitLocker base settings](./screenshots/08-bitlocker-base-settings.png)

### OS drive recovery settings

| Setting | Value |
|---------|-------|
| Choose how BitLocker-protected OS drives can be recovered | Enabled |
| Save recovery information to AD DS / Entra ID | True |
| Do not enable BitLocker until recovery information is stored | **True** |
| Configure user storage of recovery information | Require 48-digit recovery password |
| Configure storage of recovery information | Store recovery passwords only |
| Omit recovery options from the BitLocker setup wizard | True |
| Fixed / Removable data drives | Not configured |

*Verification Log — recovery and escrow settings:*

![BitLocker recovery settings](./screenshots/09-bitlocker-recovery-settings.png)

### Assignments

Included groups: `grp-ims-it-admins`, `grp-ims-supervisors`, `grp-ims-standard-users` — same three groups as the compliance policy.

*Verification Log — assignments and created policy:*

![BitLocker assignments](./screenshots/10-bitlocker-assignments.png)

![BitLocker policy created](./screenshots/11-bitlocker-policy-created.png)

> **Design Decision — Silent encryption:** With "Allow Warning For Other Disk Encryption" disabled, encryption starts without any user prompt. Users never see, print, or handle a recovery key. Under the old Ansible setup, disk encryption depended on per-machine manual setup; here it is a background consequence of enrollment.

> **Design Decision — Escrow before encryption:** "Do not enable BitLocker until recovery information is stored" set to True means a recovery key exists in Entra ID *before* the disk is encrypted. The failure mode this prevents: a disk encrypted with a key that was never successfully backed up — an unrecoverable device. IT retrieves keys from Entra ID (device object > Recovery keys); users are never a link in the key-management chain.

> **Design Decision — OS drives only:** Fixed and removable data drive encryption left unconfigured. Fleet data lives on OS drives and SharePoint/OneDrive; removable-media policy is a decision for the production migration, not this build.

---

## Windows Hello for Business

*(pending)*

---

## Update Ring

*(pending)*

---

*Last updated: July 2026*
