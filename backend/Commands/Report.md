# Command: Report

## Purpose
Generate project or business report for a specified interval.

## Inputs
- Scope:
  - one project
  - multiple projects
  - business-wide
- Date interval (`from`, `to`)
- Optional focus topics (risk, outgoing, evidence, timeline)

## Steps
1. Confirm reporting scope and date interval.
2. Read relevant sources:
   - `operations/Context/Current/ChronologyCurrent.md`
   - `operations/Context/Current/StrategyCurrent.md`
   - `operations/Context/Current/FileAnalysisCurrent.md`
   - archive files if requested or if interval crosses archived periods
3. Filter by:
   - project tag(s)
   - date interval
4. Build report sections:
   - Executive summary
   - Timeline highlights
   - Decisions and strategy changes
   - File/evidence updates
   - Outgoing communications
   - Open risks and next actions
5. Include exact references to IDs and file paths.
6. Save report to user-requested location or return inline if not specified.

## Output Template
- Scope and interval
- Key outcomes
- Detailed sections
- Pending items
