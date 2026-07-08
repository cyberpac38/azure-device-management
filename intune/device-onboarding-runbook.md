# Device Onboarding Runbook

Repeatable procedure for onboarding one fleet device, written from the pilot (VM-STAFF01 / IMS-38106). Every step below was executed and verified at least once. Batch size: 3 devices per session.

## Prerequisites (once per session)

- VMware Workstation Pro open, default VM location set to `C:\VMs` (never OneDrive)
- Windows 11 Pro ISO path known (`C:\Users\<host-user>\Downloads\Win11_25H2_English_x64.iso`)
- VM encryption password available (same one for every VM)
- Admin center open on the host: reset the target user's password before their enrollment (Users > Active users > user > Reset password), note the temp password
- Phone with Microsoft Authenticator ready for the user's MFA registration

## Per-Device Procedure

### 1. Create the VM

1. Create a New Virtual Machine > Typical
2. **"I will install the operating system later"** (never point the wizard at the ISO - Easy Install breaks the OOBE flow)
3. Guest OS: Microsoft Windows > Windows 11 x64
4. Name: `VM-<USER>` (example: VM-SUPERVISOR02), location `C:\VMs\VM-<USER>`
5. Encryption: "Only the files needed to support a TPM", the shared password, tick Remember
6. Disk: 60 GB, split files > Finish
7. Edit virtual machine settings:
   - Memory: 8192 MB (4096 minimum)
   - Processors: 2
   - CD/DVD: Use ISO image file > browse to the Windows 11 ISO
   - Hardware tab shows Trusted Platform Module: Present
   - Options > Advanced > Firmware: UEFI with **Enable secure boot ticked**

### 2. Install Windows

1. Power on > click inside the VM window immediately > press Enter at "Press any key to boot from CD" (miss it: Esc > Boot Manager > EFI VMware Virtual SATA CDROM Drive)
2. Install now > **I don't have a product key** > **Windows 11 Pro** > accept > Custom
3. Select the unallocated 60 GB disk > Next
4. Hands off through installs and reboots (~15 min). Do NOT press a key on the reboots.
5. Stop at the country/region OOBE screen.

### 3. Disconnect the ISO (pilot lesson - do this NOW)

VM > Settings > CD/DVD: untick **both** "Connected" and "Connect at power on". BitLocker refuses silent encryption while bootable media is attached (Event 853). Doing this at OOBE, before enrollment, prevents the pilot's failure entirely.

### 4. Register the hardware hash

At the country/region screen: click inside the VM, Shift+F10 > type `powershell` > Enter, then:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force
Install-Script -Name Get-WindowsAutoPilotInfo -Force
Get-WindowsAutoPilotInfo.ps1 -Online -GroupTag "IMS-Fleet"
```

- Answer `Y` to the NuGet prompt
- Sign in as the tenant admin when the window appears; choose **"No, this app only"** on the stay-signed-in prompt
- Wait for **"1 devices imported successfully"** then **"All devices synced"** (~2-4 min)
- The execution policy line must be re-run in every new PowerShell session - it does not persist

### 5. Wait for profile assignment

Portal: Devices > Enrollment > Windows > Windows Autopilot > Devices. The new serial (VMware-...) appears with group tag IMS-Fleet. Wait for Profile status: **Assigned** (5-30 min - normal propagation, not a fault). Build the next VM while waiting.

### 6. Enroll as the user

1. VM > Power > Reset (OOBE must restart to pull the Autopilot profile)
2. Answer country/keyboard normally
3. Organization sign-in screen appears (not the consumer setup - if consumer setup appears, the profile was not Assigned yet; reset again once it is)
4. Sign in as the device's user (staff02@..., etc.) with the temp password > forced password change > MFA registration via Authenticator > ESP runs > mandatory Hello PIN > desktop
5. Record the user's new password and PIN in the offline credentials sheet (never in this repo)

### 7. Verify

Portal: Devices > Windows > the new IMS-xxxxx device:

- Primary user is the correct account, ownership Corporate
- Recovery keys: a BitLocker key ID appears within ~30 min of desktop (never click Show Recovery Key while capturing screenshots)
- Device compliance: grace period at first, all checks Compliant within a couple of sync cycles / one reboot (Secure Boot and BitLocker rows lag - attestation, not failure)

After verification the VM stays powered off except a weekly boot so it checks in and holds compliance.

## Fleet Tracking

| Device | User | Hash imported | Assigned | Enrolled | Key escrowed | Compliant |
|--------|------|--------------|----------|----------|--------------|-----------|
| VM-STAFF01 | staff01 | Done | Done | Done (IMS-38106) | Done | Done |
| VM-STAFF02 | staff02 | | | | | |
| VM-STAFF03 | staff03 | | | | | |
| VM-STAFF04 | staff04 | | | | | |
| VM-STAFF05 | staff05 | | | | | |
| VM-STAFF06 | staff06 | | | | | |
| VM-SUPERVISOR01 | supervisor01 | | | | | |
| VM-SUPERVISOR02 | supervisor02 | | | | | |
| VM-SUPERVISOR03 | supervisor03 | | | | | |
| VM-ITSADMIN01 | itsadmin01 | | | | | |
| VM-ITSADMIN02 | itsadmin02 | | | | | |

When all 11 rows are complete: flip CA-003 (require compliant device) from report-only to On.

---

*Last updated: July 2026*
