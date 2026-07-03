# IMS Azure Device Management

I managed 11 Windows workstations at a small business using Ansible, an Ubuntu server, and WinRM. It works — but everything breaks if a machine is off the local network, and onboarding takes 2 hours per device.

This project moves that setup to Microsoft Azure. Built on the real office infrastructure, documented as I go.

---

## The Problem

- Machines have to be on-site to receive updates or config changes
- Onboarding requires physical access and 6 Ansible playbooks
- No centralised identity — everyone has local accounts
- No remote wipe, no compliance reporting, no cloud backup

Fine for 11 machines. Breaks at 30.

---

## What I am Replacing It With

| Before (Ansible) | After (Azure) |
|-----------------|---------------|
| Ansible playbooks | Microsoft Intune |
| Local user accounts | Microsoft Entra ID |
| 6-playbook onboarding | Windows Autopilot |
| UFW firewall rules | Conditional Access |
| Manual update scheduling | Windows Update for Business |
| SMB file share (S: drive) | SharePoint / OneDrive |

---

## Build Progress

| Phase | Focus | Status |
|-------|-------|--------|
| 1 | Azure account + GitHub repo + security baseline | Complete |
| 2 | Entra ID - identity, SSPR, Conditional Access | Complete |
| 3 | Intune - compliance policies + configuration profiles | In Progress |
| 4 | Device enrollment - Autopilot + Entra join | Pending |
| 5 | Monitoring - Log Analytics + Azure Monitor alerts | Pending |
| 6 | Terraform - infrastructure as code | Pending |

---

## Target Architecture

Microsoft Entra ID
  - MFA enforced for all users
  - Legacy authentication blocked
  - Compliant device required (Report-only pending Intune enrollment)

Microsoft Intune
  - Compliance policies
  - Configuration profiles
  - BitLocker + Windows Hello for Business

Windows 11 Pro Fleet - 35 devices (current: 11)
  - Entra ID joined
  - Autopilot enrolled
  - Defender for Endpoint

---

## Repo Structure

Current structure reflects completed phases only:

  azure-device-management/
  README.md
  .gitignore
  entra-id/
    emergency-access-account.md
    conditional-access-policies.md
    users-and-groups.md
    sspr.md
    screenshots/

Folders for intune/, monitoring/, and terraform/ will be added as those phases are completed.

---

## Security

- Break-glass emergency access account created and excluded from all CA policies
- Legacy authentication blocked from day one (CA-002 set to On)
- MFA enforced via Microsoft-managed and user-created Conditional Access policies
- No credentials or tenant identifiers stored in this repository

---

Martin Nurse - Cloud System and Quality Assurance Engineer
AZ-900 (June 2026) | AZ-104 in progress
