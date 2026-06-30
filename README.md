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

## What I'm Replacing It With

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

| Month | Focus | Status |
|-------|-------|--------|
| 1 | AZ-900 + Azure foundations + GitHub setup | ✅ Complete |
| 2 | AZ-104 + Entra ID + VNet + NSG + Bastion | 🔄 In Progress |
| 3 | Terraform — infrastructure as code | ⏳ Pending |
| 4 | CI/CD pipeline — GitHub Actions | ⏳ Pending |
| 5 | Portfolio + LinkedIn | ⏳ Pending |
| 6 | Intune + Autopilot capstone deployment | ⏳ Pending |

---

## Target Architecture

```
[ Microsoft Entra ID ]
     | No standing admin access (PIM)
     | MFA + compliant device required
     |
[ Microsoft Intune ]
     | Compliance policies
     | Configuration profiles
     | BitLocker + Windows Hello for Business
     |
[ Windows 11 Pro Fleet — 35 devices ]
     | Entra ID joined · Autopilot enrolled
     | Defender for Endpoint
```

---

## Security

- No standing admin privilege — PIM required to activate roles
- Legacy authentication blocked from day one
- Terraform state kept private — no public storage access
- CI/CD authenticates via OIDC — no stored secrets in GitHub
- Decisions documented in SECURITY.md

---

## Repo Structure

```
ims-azure-device-management/
├── README.md
├── SECURITY.md
├── architecture/
├── entra-id/
├── intune/
├── terraform/
├── .github/workflows/
└── scripts/
```

---

Martin Nurse — Cloud System & Quality Assurance Engineer  
AZ-900 (June 2026) | AZ-104 in progress
