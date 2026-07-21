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

### 4. Register the hardware hash (file method - the reliable path)

The `-Online` interactive registration does NOT work in this pilot environment: the Web Account Manager (WAM) sign-in window opens hidden behind OOBE and the upload silently does nothing, leaving the device unregistered with no error. The reliable path is to export the hash to a file and import it in the portal - no interactive sign-in involved.

At the country/region screen: click inside the VM, Shift+F10 > type `powershell` > Enter. Run these in the SAME window, and type the third line in full (the execution-policy bypass dies with the session, and Tab-completion mangles the script name to `Get-WindowsAutoPilot.ps1`):

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force
Install-Script -Name Get-WindowsAutoPilotInfo -Force
Get-WindowsAutoPilotInfo.ps1 -OutputFile C:\HWID.csv -GroupTag "IMS-Fleet"
```

- Answer `Y` to the NuGet prompt; wait for a clean prompt before the third line
- Success looks like `Gathered details for device with serial number: VMware-...`; confirm the file with `dir C:\HWID.csv`
- Type the `-GroupTag` value carefully - a casing slip like `IMS-fLEET` still assigns (Entra string-value comparison is case-insensitive) but displays inconsistently against the other devices

Move the file to the host and import it:

1. Attach a USB drive to the VM: VM > Removable Devices > [drive] > Connect (Connect to a virtual machine)
2. Find its drive letter with `Get-Volume`, then `copy C:\HWID.csv <USBletter>:\`
3. Return the USB to the host: VM > Removable Devices > [drive] > Disconnect (Connect to Host)
4. Portal: Devices > Enrollment > Windows > Windows Autopilot > Devices > **Import** > select HWID.csv > Import > **Sync**

The group tag travels inside the CSV, so no manual tagging is needed. Import processing lags ~10-15 minutes before the device appears - Sync and Refresh, do not assume it failed early.

> **Field note - why not -Online:** `-Online` needs an interactive Microsoft Graph sign-in through WAM, whose window hides behind the OOBE screen and fails silently here. The file export needs no sign-in and cannot fail that way. This replaced the original `-Online` procedure after it cost a full session of retries on the first batch.

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
| VM-STAFF01 | staff01 | Done | Done | Done (IMS-65395) | Done | Done |
| VM-STAFF02 | staff02 | | | | | |
| VM-STAFF03 | staff03 | | | | | |
| VM-STAFF04 | staff04 | | | | | |
| VM-STAFF05 | staff05 | | | | | |
| VM-STAFF06 | staff06 | | | | | |
| VM-SUPERVISOR01 | supervisor01 | Done | Done | Done (IMS-78938) | Done | Done |
| VM-SUPERVISOR02 | supervisor02 | | | | | |
| VM-SUPERVISOR03 | supervisor03 | | | | | |
| VM-ITSADMIN01 | itsadmin01 | Done | Done | Done (IMS-84200) | Done | Done |
| VM-ITSADMIN02 | itsadmin02 | Done | Done | Done (IMS-98067) | Done | Done |

When all 11 rows are complete: flip CA-003 (require compliant device) from report-only to On.

---

*Last updated: July 2026*
