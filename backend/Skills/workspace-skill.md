# Universal Business Workspace Skill

## Scope

This skill applies to all work inside `Workspaces/Template/` and its subfolders.

## Workspace Intent

This workspace is business-wide, not single-project.
It supports multiple concurrent projects while preserving strict auditability.
It combines:
- project governance patterns from the existing MDC rules
- reusable command patterns inspired by command-first workflows

## Mandatory Startup Sequence (Always)

Before doing any task:
1. Read `operations/Context/Current/General.md`.
2. Read:
   - `operations/Context/Current/ChronologyCurrent.md`
   - `operations/Context/Current/StrategyCurrent.md`
   - `operations/Context/Current/FileAnalysisCurrent.md`
3. Output a short startup summary:
   - business name
   - active projects
   - active tasks per project
   - top current risks
4. List every active project from `General.md`'s "Active Projects Register" (name + priority).
5. Ask: "Which project do you have an update for or want to continue working on?" Offer "None — general workspace tasks" as an option. If user already stated a project, skip this prompt and use that project.

## Command Dispatch Rule

When user asks to run a named command, read and execute the matching file in `backend/Commands/`:
- `Prime` -> `backend/Commands/Prime.md`
- `Sweep` -> `backend/Commands/Sweep.md`
- `Screen` -> `backend/Commands/Screen.md`
- `Report` -> `backend/Commands/Report.md`
- `Archive` -> `backend/Commands/Archive.md`
- `Create-Plan` -> `backend/Commands/Create-Plan.md`
- `Implement` -> `backend/Commands/Implement.md`
- `PermitAll` -> `backend/Commands/PermitAll.md`
- `Transcribe` -> `backend/Commands/Transcribe.md`
- `US` -> `backend/Commands/US.md`
- `EmailRead` -> `backend/Commands/EmailRead.md`
- `EmailDraft` -> `backend/Commands/EmailDraft.md`
- `EmailSent` -> `backend/Commands/EmailSent.md`
- `Setup` -> `backend/Commands/Setup.md`
- `NewProject` -> `backend/Commands/NewProject.md`

If a command name is ambiguous, ask one clarifying question.

## Supplementary Skill Loading

Additional skill files in `backend/Skills/` extend this workspace skill with specialised protocols.
The MDC file lists each supplementary skill and its trigger condition.

Rules:
- Supplementary skills extend but never override this file.
- If a conflict exists between a supplementary skill and this file, this file wins unless the user explicitly overrides.
- The agent reads a supplementary skill only when its trigger condition (defined in the MDC) matches the current task.
- Multiple supplementary skills may be active at the same time if multiple triggers match.

Current supplementary skills:
- `backend/Skills/llm-enhanced-ocr.md` — extends the OCR Pipeline section
- `backend/Skills/safety-guard.md` — extends the Safety and Escalation Rules section
- `backend/Skills/web-research.md` — extends the Deterministic Filing Table for research
- `backend/Skills/draft-governance.md` — extends the Outgoing Governance section
- `backend/Skills/video-analysis.md` — extends workspace with YouTube video visual analysis

## Core File System Contract (6+1)

Current context:
- `operations/Context/Current/General.md`
- `operations/Context/Current/ChronologyCurrent.md`
- `operations/Context/Current/StrategyCurrent.md`
- `operations/Context/Current/FileAnalysisCurrent.md`

Archive context:
- `operations/Context/Archive/ChronologyArchive.md`
- `operations/Context/Archive/StrategyArchive.md`
- `operations/Context/Archive/FileAnalysisArchive.md`

Rules:
- `General.md` is never archived.
- Current files store active project data.
- Archive files store completed project data moved by `Archive` command.
- Every project-scoped entry in Current files must include explicit project tag(s):
  - `[Project: <Name>]`
- Multi-project relevance is allowed in one entry.

## Required Workspace Directories

- `operations/Inbox/`
- `operations/Outbox/`
- `operations/Drafts/`
- `operations/Operations/OCR/`
- `operations/Operations/Transcripts/`
- `operations/Records/`
- `operations/Research/`
- `operations/Admin/`
- `backend/Commands/`
- `backend/Skills/`
- `backend/Scripts/`

If a required directory is missing, create it before filing work products.

## Global Conventions (Always Use)

- ISO date format: `YYYY-MM-DD`
- Stable IDs:
  - `DOC-###` for incoming documents
  - `EMAIL-###` for email/message artifacts
  - `OUT-###` for outgoing artifacts
  - `ISSUE-###` for gaps/issues
  - `TASK-###` or `ACT-###` for actions
- Exact file paths in backticks
- Consistent naming for people/entities
- No duplicate storage when filing (move, do not copy), unless explicitly requested

## Intake Sweep Protocol (Business-Wide)

Before substantive analysis or drafting, run intake sweep when:
- new files were dropped in root or major folders
- user requests filing, cleanup, or "update workspace"
- data volume is high and filing state is unclear

### Sweep Discovery Scope

Check:
- `Workspaces/Template/` root
- `operations/Inbox/`
- `operations/Drafts/`
- `operations/Operations/`
- user-specified staging folders

### Unfiled Detection Rule

Unfiled means:
- file appears in a location inconsistent with its type or lifecycle
- screenshot-like files without sibling transcript
- incoming files in generic holding paths rather than dated packs

### Always-Ask Project Routing Rule

When filing any item that is project-specific:
- always ask which project(s) it belongs to
- do not auto-route silently
- allow multiple project tags for one dataset

If the user does not know:
- file temporarily with `[Project: Unknown]`
- ask for follow-up classification at earliest safe point

### Intake Pack Naming

- Incoming packs: `operations/Inbox/<YYYY-MM-DD>_From_<Source>/`
- Outgoing packs: `operations/Outbox/<YYYY-MM-DD>_<Description>/`
- If date missing, use current date
- If source unknown, use `Unknown` and ask once

## Deterministic Filing Table

Use this default routing:

- screenshot/photo of written exchange (`png/jpg/webp/gif`, screenshot-like PDF):
  - `operations/Inbox/<YYYY-MM-DD>_From_<Source>/`
  - plus transcript workflow
- incoming originals (`eml/msg/pdf/xlsx/csv/txt`):
  - `operations/Inbox/<YYYY-MM-DD>_From_<Source>/`
- drafts (`docx/md` not confirmed sent):
  - `operations/Drafts/`
- confirmed sent output:
  - `operations/Outbox/<YYYY-MM-DD>_<Description>/`
- scripts (`py/ps1/sh`):
  - `backend/Scripts/`
- policy/research docs:
  - `operations/Research/`
- contracts/evidence/records:
  - `operations/Records/`
- admin/contacts/invoices/fee docs:
  - `operations/Admin/`
- ambiguous:
  - file to `operations/Inbox/<YYYY-MM-DD>_From_Unknown/`
  - ask one clarifying question

## Screenshot and Transcript Protocol

### Two-Phase Chat Screenshot Intake

For screenshot provided in chat with no saved file path yet:

Phase A:
- create destination pack
- create sibling transcript `<basename>_transcript.txt` with fields:
  - CapturedDate
  - Medium
  - Source
  - OriginalFile
  - Notes
- set `OriginalFile: CHAT_ATTACHMENT_PENDING`
- add `Retention: ScreenshotDeletedAfterTranscription (chat attachment)` in Notes
- add chronology entry that screenshot was captured

Phase B:
- after user saves image, move it into destination pack (provenance step)
- update transcript `OriginalFile` to real filename
- add `OriginalFileStatus: DeletedAfterTranscription`
- delete image file after transcript update (chat-origin only)

### On-Disk Screenshot Intake

If screenshot/photo was already on disk at discovery time:
- file it into pack
- keep original image after filing

### Transcript Completeness Sweep

During `Sweep`, if screenshot-like file exists without sibling `_transcript.txt`:
- create transcript file with required fields
- add one-line chronology note with storage path

## Outgoing Governance: Drafts -> Outbox

Rules:
1. Keep all outgoing work in `operations/Drafts/` until user explicitly confirms sent.
   - This includes email and letter drafts. When the user asks to "draft" something, "save a draft," or "put it in drafts," always use `operations/Drafts/` — never create a root-level `Drafts/` folder, and never use Gmail drafts unless the user explicitly says "Gmail draft."
   - Drafts must always be saved in Word format (.docx) generated via python-docx. Follow the same naming convention as Outbox packs (e.g. `OUT-<proj>-###_<Description>.docx`).
2. Never assume sent.
3. After confirmation (user says "sent", "process as sent", "it's sent", "mark as sent", or similar):
   - move files to `operations/Outbox/<YYYY-MM-DD>_<Description>/`
   - **delete only the specific draft folder that was confirmed sent** from `operations/Drafts/` — do not touch other drafts
   - update `ChronologyCurrent.md` with sent event
   - update `FileAnalysisCurrent.md` `OUT-###` status to `Sent/Locked`
   - update `StrategyCurrent.md` action/risk/change-log references
4. After any draft deletion, check `operations/Drafts/` for empty subfolders and remove them.

Locked-file retry protocol is not mandatory in this workspace.

## OCR Pipeline for Scanned PDFs

When new PDF is filed:
1. Check embedded text with PyMuPDF (`fitz`) page text extraction.
2. If all pages are whitespace-only:
   - run Tesseract OCR at 300 DPI
   - concatenate page outputs with clear page separators
3. Save OCR output to:
   - `operations/Operations/OCR/<original_filename>_OCR.txt`
4. Update analysis entry with OCR path.
5. If OCR fails:
   - record failure in file analysis
   - set manual transcription follow-up action.

Handwritten content handling:
- attempt OCR first
- if low-quality extraction, mark as partial and request manual verification

## Audio Transcription Pipeline (faster-whisper CUDA)

For audio/video transcription tasks, run the **Transcribe** command (`backend/Commands/Transcribe.md`).

The pipeline uses faster-whisper on CUDA with selectable quality profiles (Light → Maximum), 2-pass gap-fill methods, speaker diarization via pyannote.audio, mandatory monitoring, and automated QC loops. It produces `.txt`, `.srt`, and `.json` outputs with word-level timestamps.

See the Transcribe command for full workflow details.

## EML/ZIP Evidence-Pack Semantics

### EML

When handling `.eml`:
- capture sender, recipient(s), subject, date (if available)
- list all attachments
- each attachment gets its own analysis entry with its own ID/path
- link attachment entries back to parent email entry

### ZIP

When handling `.zip`:
- list all contained files
- extract file set to appropriate folders
- create archive entry for the zip itself
- create separate entries for significant extracted files (especially PDFs)

### Missing Referenced Attachment

If an attachment is referenced but unavailable:
- create placeholder entry with status:
  - `Missing - Re-forward required`
- record that analysis is blocked pending retrieval

## Current/Archive Lifecycle Rules

### Active Project Tracking

`General.md` must track:
- active projects
- current tasks
- recently completed projects

Whenever project status changes:
- update `General.md` same day
- update chronology with status event

### Archive Command Behavior

When archiving a completed project:
1. Move project-tagged entries from each Current file to matching Archive file.
2. Preserve IDs, dates, paths, and cross references.
3. Leave tombstone note in `ChronologyCurrent.md`.
4. Move project row from Active to Recently Completed in `General.md`.
5. Do not archive business-general data.
6. For multi-project entries:
   - if any tagged project remains active, keep entry in Current
   - optionally duplicate summary in Archive with pointer to Current

## No Information Lost Protocol

When restructuring:
1. Freeze source snapshot first (copy to safe location or archive section).
2. Move strategy content first.
3. Move file analysis content second.
4. Move chronology content last.
5. Run no-loss audit:
   - every meaningful section must exist in Current, Archive, or explicitly deprecated notes
6. If uncertain where item belongs, default to file analysis with chronology cross-reference.

## Large Dataset Policy (UltraSearch)

When dataset is large, always use UltraSearch before deep reads, even without explicit `US` command.

Executable:
- `C:\Tools\UltraSearch\cli.exe`

Common usage:
- filename search: `cli.exe search "<term>"`
- content search: `cli.exe search --content "<term>"`
- filtered search: `cli.exe search "<term>" --ext pdf --after 2025-01-01`
- status: `cli.exe status`

After search:
- verify paths before analysis
- note query terms and key result paths in chronology

## Planning and Implementation Modes

### Create-Plan

For requested changes:
- produce step-by-step plan
- list targeted files
- list open questions/assumptions
- do not implement until user confirms final plan

### Implement

After user confirms plan:
- execute steps in sequence
- update documentation as changes land
- report completion and any residual risks

## PermitAll Mode

When user requests full permission mode:
- acknowledge elevated permission handling
- proceed without per-step approval prompts
- still avoid destructive actions unless explicitly requested

## Safety and Escalation Rules

- Never delete or move user data irreversibly without explicit workflow justification.
- For ambiguous classification, ask user.
- If requested behavior seems to drop existing functionality from prior systems, ask:
  - "Is this omission intentional or accidental?"
- If conflict exists between command file and this skill:
  - prefer explicit user instruction
  - otherwise prefer this skill
  - record the conflict note in chronology.

## Minimal Daily Hygiene

At end of substantial work session:
- ensure `General.md` status is current
- ensure chronology has dated event trail
- ensure strategy and file analysis references are synchronized
- ensure Drafts/Outbox state is accurate

This skill is the authoritative behavioral specification for the `Workspaces/Template/` workspace.
