# Notebook Analysis Specs

## ADDED Requirements

### Requirement: Python Environment Configuration
The project MUST provide a dependency definition file for the analysis environment.

#### Scenario: Setup Analysis Environment
- **GIVEN** a user has cloned the repository
- **WHEN** they run `pip install -r requirements.txt`
- **THEN** the necessary libraries (`pandas`, `jupyter`, `python-evtx`, etc.) are installed successfully.

### Requirement: Interactive Log Analysis
The project MUST provide a Jupyter Notebook capable of parsing standard `Get-BitLockerStatus.ps1` output.

#### Scenario: Parse BitLocker Volume Text
- **GIVEN** a `Get-BitLockerVolume.txt` file in the data directory
- **WHEN** the notebook execution reaches the Volume Parsing cell
- **THEN** a Pandas DataFrame is displayed showing DriveLetter, ProtectionStatus, and EncryptionPercentage.

#### Scenario: Parse MDM XML
- **GIVEN** a `MDM/BitlockerMDM.xml` file exists
- **WHEN** the notebook execution reaches the MDM Parsing cell
- **THEN** the XML nodes are parsed into a table showing Policy names and current values.
- **AND** missing files are handled gracefully with a warning instead of a crash.

#### Scenario: Parse Event Logs
- **GIVEN** a `Microsoft-Windows-BitLocker-API_Management.evtx` file
- **WHEN** the notebook execution reaches the Event Log cell
- **THEN** the EVTX is parsed to show a timeline or table of recent Errors and Warnings.

### Requirement: Documentation
The project MUST document how to use the notebook.

#### Scenario: Read Usage Instructions
- **GIVEN** the `README.md`
- **WHEN** a user reads the Analysis section
- **THEN** they understand how to install Python, launch Jupyter, and point the notebook to their log folder.
