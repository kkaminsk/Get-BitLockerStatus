# Design: Jupyter Analysis Integration

## Architecture
The solution introduces an offline analysis layer decoupled from the collection script.

### Component Interaction
1.  **Collection**: `Get-BitLockerStatus.ps1` runs on the target machine (requires Admin). Output: Folder with `.txt`, `.xml`, `.evtx`.
2.  **Transport**: Folder is zipped (optional) and moved to an analysis machine (can be the same machine).
3.  **Analysis**:
    - **Python Environment**: Managed via `pip` and `requirements.txt`.
    - **Jupyter Notebook**: Reads files from a defined `DATA_DIR`.
    - **Libraries**:
        - `pandas`: Data manipulation and tabular display.
        - `matplotlib`/`seaborn`: Visualization of event timelines.
        - `lxml` / `xml.etree`: Parsing `BitlockerMDM.xml`.
        - `python-evtx`: Direct parsing of `Microsoft-Windows-BitLocker-API_Management.evtx` and `system.evtx`.

### Data Flow
`[Log Folder]` -> `[Analysis.ipynb]` -> `[Pandas DataFrames]` -> `[HTML/Markdown Report]`

## Considerations
- **Dependencies**: We must minimize heavy dependencies where possible, but `pandas` is essential for this feature.
- **Path Handling**: The notebook must robustly handle relative paths to find the log data, regardless of where the notebook is saved relative to the data.
- **Error Handling**: The notebook cells should gracefully handle missing files (e.g., if `-MDM` wasn't used, `BitlockerMDM.xml` won't exist).
