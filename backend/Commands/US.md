# Command: US

## Purpose
Run UltraSearch for large dataset discovery before deep reading.

## Executable
- `C:\Tools\UltraSearch\cli.exe`

## Inputs
- Query term(s)
- Optional mode:
  - filename
  - content
- Optional filters:
  - extension
  - date range
  - size

## Steps
1. Confirm query intent and search mode.
2. Run appropriate UltraSearch command:
   - filename search:
     - `cli.exe search "<term>"`
   - content search:
     - `cli.exe search --content "<term>"`
   - filtered:
     - `cli.exe search "<term>" --ext pdf --after 2025-01-01`
3. Verify returned paths exist before using results.
4. Summarize top matches with paths.
5. If used in a project workflow, record query summary in chronology.

## Output
- Search command(s) used
- top matches
- recommended next files to inspect
