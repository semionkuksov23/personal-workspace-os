# Command: Archive

## Purpose
Move completed project data from Current context files to Archive context files.

## Inputs
- Project name (required)
- Completion date (required)
- Archive reason/outcome (optional but recommended)

## Steps
1. Confirm target project and completion date.
2. Read Current files:
   - `operations/Context/Current/General.md`
   - `operations/Context/Current/ChronologyCurrent.md`
   - `operations/Context/Current/StrategyCurrent.md`
   - `operations/Context/Current/FileAnalysisCurrent.md`
3. Select entries tagged with `[Project: <Name>]`.
4. For multi-project entries:
   - keep in Current if any other tagged project is active
   - optionally add archive summary pointer.
5. Move selected entries to:
   - `operations/Context/Archive/ChronologyArchive.md`
   - `operations/Context/Archive/StrategyArchive.md`
   - `operations/Context/Archive/FileAnalysisArchive.md`
6. Preserve IDs, dates, and original references.
7. Add chronology tombstone in Current:
   - `[ARCHIVED] Project <Name> moved to archive files on <YYYY-MM-DD>`
8. Update `operations/Context/Current/General.md`:
   - remove from active projects
   - add to recently completed projects with archive references
9. Do not archive business-general non-project data.
10. Run no-loss audit and report moved entry counts.

## Output
- Archive summary with counts, remaining multi-project items, and pointers.
