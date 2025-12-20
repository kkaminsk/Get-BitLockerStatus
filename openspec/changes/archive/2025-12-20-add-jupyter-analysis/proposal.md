# Add Jupyter Notebook Analysis

## Summary
Introduce a Jupyter Notebook (`Analysis.ipynb`) and Python dependency management (`requirements.txt`) to enable rich analysis and visualization of the diagnostic data collected by `Get-BitLockerStatus.ps1`. This allows support engineers and administrators to interactively explore logs, XML data, and registry exports without manual parsing.

## Why
The current output consists of raw text, XML, and EVTX files. While comprehensive, it requires manual inspection or separate tools to correlate data. A Jupyter Notebook provides an interactive environment to:
- Parse and display structured data (BitLocker status, TPM info).
- Visualize event log frequency.
- Compare MDM policies against baselines.
- Provide a "runnable" analysis report for collected artifact bundles.

## What Changes
1.  **New Artifacts**:
    - `Analysis.ipynb`: A sample notebook containing parsers and visualization logic.
    - `requirements.txt`: Python dependencies (`pandas`, `lxml`, `python-evtx`, etc.).
    - `README.md` (Update): Instructions on how to set up the Python environment and run the notebook.

2.  **Workflow**:
    - Users run `Get-BitLockerStatus.ps1` to generate logs.
    - Users launch Jupyter, point the notebook to the output folder (or copy the notebook into the folder), and run cells to analyze.

## Alternatives Considered
- **PowerShell-based Analysis**: Create a `Analyze-BitLockerStatus.ps1`. Rejected because it lacks the rich visualization and interactive data exploration capabilities of Jupyter/Pandas.
- **Web App**: Build a dedicated log viewer web app. Rejected as too heavy-weight for the current scope.
