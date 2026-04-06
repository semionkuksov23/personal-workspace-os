# Command: Sweep

## Purpose
Run deterministic intake and filing across the workspace.

## Preconditions
- Read `operations/Context/Current/General.md` and confirm active projects list.

## Discovery Scope
- `Workspaces/Template/` root
- `operations/Inbox/`
- `operations/Drafts/`
- `operations/Operations/`
- any user-specified staging folder

## Steps
1. Identify unfiled or misfiled items.
2. For each project-specific item, ask which project(s) it belongs to.
3. Route each item using deterministic mapping:
   - incoming docs -> `operations/Inbox/YYYY-MM-DD_From_Source/`
   - drafts -> `operations/Drafts/`
   - sent confirmations -> `operations/Outbox/YYYY-MM-DD_Description/`
   - scripts -> `backend/Scripts/`
   - records -> `operations/Records/`
   - research -> `operations/Research/`
   - admin -> `operations/Admin/`
4. Move files (do not duplicate) unless user requests otherwise.
5. Assign stable IDs (DOC/EMAIL/OUT/ISSUE/TASK).
6. Update:
   - `operations/Context/Current/ChronologyCurrent.md`
   - `operations/Context/Current/FileAnalysisCurrent.md`
   - `operations/Context/Current/StrategyCurrent.md` (if priorities/risks/actions changed)
7. Run transcript completeness sweep:
   - create missing `_transcript.txt` next to screenshot-like files.
8. Run OCR detection for new PDFs:
   - if scanned, create `operations/Operations/OCR/<filename>_OCR.txt`
   - link OCR path in analysis entry.
9. For `.eml`:
   - record metadata
   - enumerate attachments
   - create per-attachment analysis entries.
10. For `.zip`:
   - enumerate contents
   - extract significant files
   - create per-file analysis entries.
11. For missing referenced attachments:
   - create placeholder with `Missing - Re-forward required`.

## Output
- Summary of files moved, IDs assigned, questions asked, and follow-ups.
