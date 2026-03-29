# Dynamic Modern Desktop Provisioning

**Environment:** Entra ID Joined | Intune Managed | Windows Autopilot

---

## Overview

This solution automates device naming and application delivery at first user login across all Hudson Automotive Group rooftops. A generic Autopilot-enrolled device is renamed to the `LOC-DE-SERIAL` standard and targeted with the correct software — all without IT touching the machine.

**Example:** `HNC-SL-12345678` → Hudson Nissan of Charleston | Sales | Last 8 of serial

---

## How It Works

```
User signs in
    └─> IME detects user session
        └─> Detection script checks device name
            └─> Provisioning script queries Entra ID
                └─> Device renamed to LOC-DE-SERIAL
                    └─> Restart
                        └─> Dynamic groups push location + department apps
```

---

## Repository Contents

| File | Purpose |
|---|---|
| `Provision-Device.ps1` | Master rename script — queries Entra ID, calculates name, renames device |
| `Detect-Rename.ps1` | Intune detection script — prevents re-run if name already meets standard |

---

## Prerequisites

| Requirement | Details |
|---|---|
| Entra ID App Registration | `Intune-Dynamic-Provisioning-Tool` with `User.Read.All` application permission |
| Admin consent granted | Required for Graph API access |
| Entra ID user attributes | `OfficeLocation`, `JobTitle`, `Department`, `CompanyName` must be populated |
| Intune Win32 app | Scripts packaged with Microsoft Win32 Content Prep Tool |

---

## Naming Convention

```
LOC - DE - SERIAL
 |     |      |
 |     |      └─ Last 8 characters of device serial number (alphanumeric only)
 |     └─ 2-letter department code
 └─ 3-letter location code
```

### Department Codes

| Code | Department |
|---|---|
| AC | Accounting |
| BO | Admin & Clerical / Executive Assistant / Office Manager |
| BO | Construction / VP / CEO / COO / Director |
| BO | Back Office / IT / Admin |
| BS | Body Shop |
| FI | Finance & Insurance |
| PT | Parts |
| SL | Sales |
| SV | Service |
| ST | Shop / Technician |
| UN | Unknown (fallback) |

### Location Codes

See [full location mapping table](https://github.com/MrRobInIT/Intune-Provisioning-Tool/blob/main/dynamic-modern-desktop-provisioning.md#8-location-based-dynamic-groups) in the detailed SOP.

---

## Intune App Configuration

| Setting | Value |
|---|---|
| App type | Windows app (Win32) |
| Install command | `powershell.exe -ExecutionPolicy Bypass -File Provision-Device.ps1` |
| Uninstall command | `powershell.exe -Command "Write-Output 'No-Op'"` |
| Detection method | Custom script — `Detect-Rename.ps1` |
| Assignment | Required — All Users |

---

## Dynamic Group Targeting

After rename and reboot, Entra ID dynamic user groups automatically push the correct apps:

- **Role-based groups** → departmental software (CRM, Accounting, Techline Connect, etc.)
- **Location-based groups** → site-specific tools (Datto RMM site tokens, etc.)

---

## Monitoring

| Check | Location in Intune |
|---|---|
| Fleet-wide rename status | Apps > Windows > [App Name] > Device install status |
| Per-device app timeline | Devices > Windows > [Device] > Managed Apps |
| Script log | `C:\ProgramData\AutomotiveProvision.log` (via Collect Diagnostics) |

---

## Related Documentation

- [Full Technical SOP --requires invite](https://github.com/MrRobInIT/Intune-Provisioning-Tool/blob/main/TECHNICAL-SOP.md)
- [Microsoft Intune Admin Center](https://intune.microsoft.com)
- [Entra Admin Center](https://entra.microsoft.com)
