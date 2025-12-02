# Project Context

## Purpose
Get-BitLockerStatus.ps1 is a PowerShell diagnostic collection tool that gathers BitLocker configuration and state information from Windows devices. It is designed to help troubleshoot BitLocker encryption, TPM, Windows Recovery Environment (WinRE) issues, Group Policy BitLocker settings (FVE), and optionally MDM diagnostics. The script produces a timestamped folder containing logs, event exports, registry data, and optional MDM diagnostics for support and troubleshooting purposes.

## Tech Stack
- PowerShell 5.1 or newer
- Windows-specific system tools:
  - BitLocker cmdlets (`Get-BitLockerVolume`)
  - `manage-bde.exe` (BitLocker Drive Encryption Tool)
  - `Get-Tpm` cmdlet
  - `reagentc.exe` (Windows Recovery Environment configuration)
  - `wevtutil.exe` (Windows Events utility)
  - `reg.exe` (Registry tool)
  - `mdmdiagnosticstool.exe` (optional, for MDM diagnostics)
  - `Compress-Archive` cmdlet (for ZIP functionality)

## Project Conventions

### Code Style
- **PowerShell Best Practices**: Use approved verbs, PascalCase for functions, strict mode enabled
- **Error Handling**: `$ErrorActionPreference = 'Stop'` with structured logging
- **Logging**: Timestamped log entries with level indicators (INFO, WARN, ERROR)
- **Functions**: Use `function` keyword, parameter validation with `[Parameter()]`, explicit typing
- **Naming Conventions**:
  - Functions: Verb-Noun format (e.g., `Test-IsAdministrator`, `Write-Log`)
  - Variables: camelCase (e.g., `$logRoot`, `$folderName`)
  - Parameters: PascalCase with type annotations (e.g., `[switch] $MDM`, `[string] $OutputPath`)
- **Comments**: Use PowerShell comment-based help (`<#!` blocks) for documentation
- **Version Requirements**: `#requires -version 5.1` directive at script start

### Architecture Patterns
- **Single-Script Design**: All functionality contained in a single `.ps1` file for portability
- **Function-Based Structure**: Helper functions for discrete operations (logging, path resolution, privilege checks)
- **Non-Interactive Execution**: Designed for automation and scripting environments
- **Output Isolation**: Creates timestamped folders to avoid conflicts and preserve history
- **Graceful Degradation**: Falls back to alternative methods when primary commands fail (e.g., event log channel fallbacks)
- **Structured Logging**: STEP markers in logs for quick triage and parsing

### Testing Strategy
- **Manual Testing**: Test on various Windows editions (Client/Server) with and without BitLocker
- **Privilege Validation**: Verify administrator privilege checks work correctly
- **Parameter Combinations**: Test all parameter combinations (`-MDM`, `-ZIP`, `-UseTemp`, `-OutputPath`)
- **Event Log Fallbacks**: Test event log exports with missing or inaccessible channels
- **Edge Cases**: Test with missing MDM tools, insufficient permissions, full disks, etc.
- **Output Validation**: Verify all expected files are created and contain valid data

### Git Workflow
- **Main Branch**: `main` is the primary branch for stable releases
- **Commit Messages**: Clear, descriptive messages following conventional commits style
- **Feature Development**: Feature branches merged to main via pull requests
- **Versioning**: Track changes through git tags if formal versioning is adopted

## Domain Context

### BitLocker Drive Encryption (FVE)
BitLocker is a Windows feature that encrypts disk volumes to protect data. Key concepts:
- **Protection Status**: Volumes can be protected, unprotected, or suspended
- **Key Protectors**: Multiple methods to unlock (TPM, recovery password, PIN, USB key)
- **Recovery Keys**: 48-digit numerical passwords for emergency access
- **Group Policy**: Registry settings under `HKLM\Software\Policies\Microsoft\FVE` control BitLocker behavior

### Trusted Platform Module (TPM)
Hardware security chip that stores BitLocker keys:
- **TPM Status**: Present, Ready, Enabled, Activated, Owned
- **TPM Version**: 1.2 or 2.0 (2.0 preferred for modern systems)
- **BitLocker Integration**: TPM can automatically unlock drives at boot

### Windows Recovery Environment (WinRE)
Pre-boot recovery environment for system repair:
- **WinRE Status**: Enabled/Disabled, location path
- **BitLocker Interaction**: WinRE must be properly configured for BitLocker recovery scenarios

### Mobile Device Management (MDM)
Enterprise device management system (Intune, etc.):
- **MDM Diagnostics**: XML reports containing policy state and configuration
- **BitLocker Policies**: MDM can enforce BitLocker encryption requirements
- **Area Nodes**: XML elements in `MDMDiagReport.xml` containing BitLocker-specific configuration

### Event Logs
Windows event logs track BitLocker operations:
- **Microsoft-Windows-BitLocker-API/Management**: BitLocker-specific events
- **System**: General system events including storage and boot issues
- **Application**: Application-level events
- **DeviceManagement-Enterprise-Diagnostics-Provider**: MDM enrollment and policy events (with `-MDM`)

## Important Constraints

### Technical Constraints
- **Administrator Privileges Required**: Script MUST run elevated or will fail immediately
- **Windows-Only**: Uses Windows-specific cmdlets and tools; not cross-platform
- **PowerShell Version**: Minimum PowerShell 5.1 required for cmdlet compatibility
- **BitLocker Module**: Requires Windows editions with BitLocker support (Pro, Enterprise, Education)
- **64-bit Awareness**: Must handle 32-bit PowerShell on 64-bit systems (use Sysnative for tools)
- **Path Length Limits**: Windows MAX_PATH considerations for deeply nested output folders
- **Disk Space**: Requires sufficient space for event logs, MDM diagnostics, and ZIP archives

### Operational Constraints
- **Non-Interactive Design**: No user prompts during execution; all configuration via parameters
- **Error Isolation**: Individual collection failures should not stop the entire script
- **Output Folder Conflicts**: Timestamped folders prevent overwrites but same-minute runs could conflict
- **Log File Locking**: Activity log must remain writable throughout script execution

### Privacy and Security Constraints
- **Sensitive Data**: Output may include recovery keys, device IDs, and system configuration
- **Access Control**: Output folders should be secured; share only with trusted parties
- **Credential Storage**: Script does not store or transmit credentials
- **Audit Trail**: Activity log provides forensic record of script operations

## External Dependencies

### Windows System Tools (Required)
- **manage-bde.exe**: BitLocker Drive Encryption command-line tool (`C:\Windows\System32`)
- **reagentc.exe**: Windows Recovery Environment configuration tool (`C:\Windows\System32`)
- **wevtutil.exe**: Windows Events command-line utility (`C:\Windows\System32`)
- **reg.exe**: Registry console tool for exporting FVE policies (`C:\Windows\System32`)

### PowerShell Cmdlets (Required)
- **Get-BitLockerVolume**: BitLocker module cmdlet for volume status
- **Get-Tpm**: TrustedPlatformModule cmdlet for TPM status
- **Compress-Archive**: Microsoft.PowerShell.Archive module (for `-ZIP` functionality)

### Optional Dependencies
- **mdmdiagnosticstool.exe**: MDM diagnostic collection tool (required for `-MDM` parameter)
  - Location: `%WINDIR%\System32` or `%WINDIR%\Sysnative` (64-bit systems)
  - Generates `MDMDiagReport.xml` and `MDMDiagReport.html`
  - Only available on MDM-enrolled or Enterprise-managed devices

### Event Log Channels
- `Microsoft-Windows-BitLocker-API/Management` (primary) or `Microsoft-Windows-BitLocker/BitLocker Management` (fallback)
- `System`
- `Application`
- `Microsoft-Windows-DeviceManagement-Enterprise-Diagnostics-Provider/Admin` (with `-MDM`)
- `Microsoft-Windows-AAD/Operational` (with `-MDM`)
- `Microsoft-Windows-Shell-Core/Operational` (with `-MDM`)

### File System
- **Base Output Paths**:
  - Documents folder: `[Environment]::GetFolderPath('MyDocuments')`
  - Windows Temp: `C:\Windows\Temp`
  - Custom: User-specified via `-OutputPath`
- **ZIP Archives**: Created in parent folder of output directory
