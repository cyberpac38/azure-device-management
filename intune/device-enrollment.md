# Device Enrollment - Autopilot Pilot (Phase 4)

## Overview

Phase 4 enrolls the fleet through Windows Autopilot: hardware hash registration → dynamic group membership → deployment profile → user-driven OOBE enrollment. This document covers the pilot device (VM-STAFF01, enrolled as staff01) - proving the full chain end-to-end before batch-enrolling the remaining 10 devices. The pilot surfaced two real failures, documented in the Troubleshooting section; that is what pilots are for.

## Test Environment

The deployment is validated on VMware Workstation Pro VMs before touching production hardware - the same pilot-then-production sequencing the real migration will follow.

| Component | Value | Why |
|-----------|-------|-----|
| Hypervisor | VMware Workstation Pro (free personal license) | Host runs Windows Home (no Hyper-V); Workstation provides full vTPM |
| VM firmware | UEFI + Secure Boot enabled | Compliance policy requires Secure Boot |
| vTPM | TPM 2.0 (VM encryption enabled) | Satisfies TPM + BitLocker requirements; Intune reports it as `TPM 2.0, VMW` |
| Guest OS | Windows 11 Pro (Home cannot Entra-join) | Matches target fleet |
| Sizing | 2 vCPU / 4-8 GB RAM / 60 GB disk | One VM per fleet user, run in batches |

> **Design Decision - VMs as pilot, not pretend hardware:** The Intune/Entra backend is identical regardless of what the endpoint runs on. Everything above the hardware - enrollment, policy, compliance, encryption, Conditional Access - is the real production configuration. What VMs cannot validate (user communication, change windows, physical rollout logistics) is explicitly out of scope for the pilot.

---

## Hardware Hash Registration

From the parked OOBE screen (Shift+F10 → PowerShell), each device registers itself with the `IMS-Fleet` group tag:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force
Install-Script -Name Get-WindowsAutoPilotInfo -Force
Get-WindowsAutoPilotInfo.ps1 -Online -GroupTag "IMS-Fleet"
```

Sign-in uses the tenant admin account; the first run requires admin consent for the Autopilot Graph app. Import takes ~2 minutes per device.

*Verification Log - hash imported and synced:*

![Autopilot hash import](./screenshots/15-autopilot-hash-import.png)

> **Field note:** `-Scope Process` means the execution-policy bypass dies with the PowerShell session - a second session needs the command re-run. The first attempt here failed with `PSSecurityException` for exactly that reason.

---

## Autopilot Plumbing

Three objects connect the group tag to an enrollment experience:

| Object | Name | Key setting |
|--------|------|-------------|
| Dynamic device group | `grp-ims-autopilot-devices` | `(device.devicePhysicalIds -any (_ -eq "[OrderID]:IMS-Fleet"))` |
| Deployment profile | `IMS-UserDriven-EntraJoin` | User-driven, Entra joined, **user account type: Standard**, name template `IMS-%RAND:5%` |
| Enrollment Status Page | Default | Progress shown, 60-min timeout, device not blocked on apps |

*Verification Log - group, ESP, and profile assignment:*

![Dynamic group](./screenshots/16-autopilot-dynamic-group.png)

![ESP settings](./screenshots/17-esp-settings.png)

![Profile assigned to device](./screenshots/18-autopilot-device-assigned.png)

> **Design Decision - Standard user, not admin:** The deployment profile makes the enrolling user a standard user. Under the legacy setup every user ran with local admin; this single setting removes that entire risk class at enrollment time.

> **Design Decision - group tag as the only hook:** Every downstream behaviour (group membership → profile → policies) derives from the `IMS-Fleet` tag set at hash import. Onboarding device #2 through #11 requires zero additional portal configuration - the definition of the Autopilot value proposition versus the 6-playbook manual onboarding it replaces.

> **Field note - propagation delay:** Profile status sat at "Pending" for ~20 minutes after import before flipping to "Assigned." This is normal Autopilot behaviour, not a fault; the device must not begin OOBE enrollment until status reads Assigned.

---

## Pilot Enrollment (VM-STAFF01 → staff01)

User-driven flow as experienced by the end user: OOBE → organization sign-in → temp password change → MFA registration → ESP provisioning → mandatory Hello PIN → desktop. Device named `IMS-38106` by the profile's naming template; staff01 attached as primary user; ownership Corporate.

*Verification Log - enrollment sequence:*

![MFA registered during OOBE](./screenshots/19-oobe-mfa-registered.png)

![ESP provisioning](./screenshots/20-esp-provisioning.png)

![Windows Hello enforced by policy](./screenshots/21-hello-pin-enforced.png)

![staff01 desktop](./screenshots/22-desktop-staff01.png)

![Device in Intune with grace-period compliance](./screenshots/23-device-enrolled-overview.png)

![vTPM 2.0 reported by Intune](./screenshots/25-hardware-vtpm.png)

> **Field note - SSPR used in production:** The tenant admin password was reset mid-phase via self-service password reset - the SSPR deployment from Phase 2 doing its job for its own administrator.

With the first device's build number known (10.0.26200.8037), the compliance policy's deferred minimum OS version was set:

![Minimum OS version set](./screenshots/26-policy-min-os-set.png)

---

## Troubleshooting Log

The pilot device landed in the compliance grace period with two failing checks - BitLocker and Secure Boot:

![Compliance state at first evaluation](./screenshots/24-compliance-grace-period.png)

![Per-setting compliance breakdown](./screenshots/28-per-setting-compliance.png)

![No recovery key escrowed](./screenshots/27-recovery-keys-empty.png)

**Diagnosis:** BitLocker-API event log on the device (Event ID 853):

> *"Failed to enable Silent Encryption. TPM is not available. Error: BitLocker Drive Encryption detected bootable media (CD or DVD) in the computer. Remove the media and restart the computer before configuring BitLocker."*

![Event 853](./screenshots/29-event-853-bitlocker-blocked.png)

**Root cause:** the Windows 11 installation ISO was still attached to the VM's virtual DVD drive. BitLocker refuses silent encryption while bootable media is present. The Secure Boot failure was stale device-health attestation from the same blocked evaluation cycle - the VM's firmware had Secure Boot enabled throughout.

**Remediation:** disconnect the virtual DVD (both "Connected" and "Connect at power on"), verify Secure Boot in VM firmware settings, reboot, force an Intune sync:

![ISO disconnected](./screenshots/30-vm-cd-disconnected.png)

![Secure Boot verified in VM firmware](./screenshots/31-vm-secure-boot-verified.png)

**Resolution:** within one restart-and-sync cycle of removing the virtual DVD, silent encryption started and the 48-digit recovery key appeared in Entra ID - key escrowed *before* the disk finished encrypting, exactly as the escrow-before-encrypt policy requires. No user action, no visible prompts.

![Recovery key escrowed to Entra](./screenshots/32-recovery-key-escrowed.png)

**Process change for the batch:** the fleet build checklist now disconnects the ISO immediately after Windows installation, before OOBE. The failure class is eliminated for devices 2-11.

**Final state:** after the next attestation cycle, all 11 compliance checks report Compliant - BitLocker and Secure Boot included. The pilot device went enrolled → grace period → diagnosed → remediated → compliant without any manual encryption steps on the device itself.

![All compliance checks green](./screenshots/33-pilot-compliant.png)

---

## Next

Batch enrollment of the remaining 10 devices (itsadmin01-02, supervisor01-03, staff02-06), then flipping CA-003 (require compliant device) from report-only to On.

---

*Last updated: July 2026*
