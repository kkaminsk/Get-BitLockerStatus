# Get-BitLockerStatus.ps1 - Execution Explained

This document explains what the `Get-BitLockerStatus.ps1` PowerShell script does when you run it.

## Purpose

The script collects diagnostic information about BitLocker encryption status, TPM (Trusted Platform Module), Windows Recovery Environment, and related event logs from a Windows computer. It bundles all this information into a timestamped folder for troubleshooting or auditing purposes.

## Prerequisites

- **PowerShell 5.1 or later** is required
- **Must run as Administrator** - the script checks this and exits with an error if not elevated

## Command-Line Options

| Parameter | Description |
|-----------|-------------|
| `-MDM` | Collects additional MDM (Mobile Device Management) diagnostic data |
| `-OutputPath` | Specifies a custom folder location for output |
| `-UseTemp` | Uses `C:\Windows\Temp` instead of Documents folder |
| `-ZIP` | Creates a ZIP archive of all collected files |

## Step-by-Step Execution

### 1. Administrator Check
The script first verifies it's running with Administrator privileges. If not, it displays an error and exits immediately.

### 2. Determine Output Location
The script decides where to save collected data:
- If `-OutputPath` is provided, it uses that location
- If `-UseTemp` is specified, it uses `C:\Windows\Temp`
- Otherwise, it defaults to the user's Documents folder

### 3. Create Output Folder
Creates a timestamped folder named `BitLockerStatus-DD-MM-YYYY-HH-MM` in the chosen location. All collected files go into this folder.

### 4. Initialize Logging
Creates a log file (`Get-BitLockerStatus.log`) that records every action the script takes, including timestamps and success/failure status.

### 5. Collect BitLocker Volume Information
- **Command**: `Get-BitLockerVolume`
- **Output**: `Get-BitLockerVolume.txt`
- **What it captures**: Detailed information about all BitLocker-protected volumes, including encryption status, protection status, key protectors, and encryption percentage.

### 6. Collect manage-bde Status
- **Command**: `manage-bde.exe -status`
- **Output**: `Manage-BDE_Status.txt`
- **What it captures**: BitLocker Drive Encryption status from the command-line tool, showing volume details, conversion status, percentage encrypted, and encryption method.

### 7. Collect TPM Status
- **Command**: `Get-Tpm`
- **Output**: `Get-TPM.txt`
- **What it captures**: Information about the Trusted Platform Module chip, including whether it's present, ready, enabled, activated, and owned.

### 8. Collect Windows Recovery Environment Status
- **Command**: `reagentc.exe /info`
- **Output**: `Reagentc.txt`
- **What it captures**: Windows Recovery Environment (WinRE) configuration, including whether it's enabled and the location of the recovery image.

### 9. Export BitLocker Event Log
- **Tool**: `wevtutil.exe`
- **Output**: `Microsoft-Windows-BitLocker-API_Management.evtx`
- **What it captures**: The Windows event log containing BitLocker-related events. The script tries two possible channel names to handle different Windows versions.

### 10. Export System Event Log
- **Tool**: `wevtutil.exe`
- **Output**: `system.evtx`
- **What it captures**: The entire System event log, which may contain relevant BitLocker, TPM, or boot-related events.

### 11. Export BitLocker Group Policy Registry Settings
- **Tool**: `reg.exe export`
- **Output**: `FVE_Policies.reg`
- **What it captures**: The registry key `HKLM\Software\Policies\Microsoft\FVE` which contains BitLocker Group Policy settings. This key may not exist if no BitLocker policies are configured.

### 12. MDM Diagnostics (Only if `-MDM` specified)
- **Tool**: `mdmdiagnosticstool.exe`
- **Output folder**: `MDM\`
- **What it captures**:
  - Runs the MDM diagnostics tool to generate comprehensive device management logs
  - Parses `MDMDiagReport.xml` to extract BitLocker-specific policy areas
  - Creates `BitlockerMDM.xml` containing only the BitLocker-related MDM configuration

### 13. Create ZIP Archive (Only if `-ZIP` specified)
- **Output**: `<COMPUTERNAME>-BitLockerStatus-DD-MM-YYYY-HH-MM.zip`
- **Location**: Parent folder of the output directory (not inside it)
- **What it does**: Compresses all collected files into a single ZIP archive for easy sharing or storage.

### 14. Completion
The script logs completion and displays the paths to:
- The output folder containing all collected files
- The activity log file
- The ZIP archive (if created)

## Output Files Summary

| File | Contents |
|------|----------|
| `Get-BitLockerVolume.txt` | PowerShell BitLocker volume details |
| `Manage-BDE_Status.txt` | Command-line BitLocker status |
| `Get-TPM.txt` | TPM chip information |
| `Reagentc.txt` | Windows Recovery Environment status |
| `Microsoft-Windows-BitLocker-API_Management.evtx` | BitLocker event log |
| `system.evtx` | System event log |
| `FVE_Policies.reg` | BitLocker Group Policy registry export |
| `MDM\BitlockerMDM.xml` | BitLocker MDM policies (with `-MDM`) |
| `Get-BitLockerStatus.log` | Script activity log |

## Error Handling

The script uses a "best effort" approach:
- If a particular tool or cmdlet isn't available, it logs a warning and continues
- If a data source doesn't exist (like the FVE registry key), it notes this and moves on
- All errors are logged to the activity log file
- The script doesn't stop on individual failures; it completes all possible steps

## 32-bit/64-bit Compatibility

The script includes special handling for running in a 32-bit PowerShell process on a 64-bit Windows system. It uses the `Sysnative` virtual folder to access 64-bit versions of:
- `wevtutil.exe`
- `reg.exe`
- `mdmdiagnosticstool.exe`

This ensures accurate data collection regardless of which PowerShell host is used.
