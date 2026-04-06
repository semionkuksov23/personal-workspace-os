# ChronologyCurrent

## Rules
- This file contains events for active projects only.
- Every entry must include at least one explicit project tag.
- Use ISO dates: `YYYY-MM-DD`.
- Use exact file paths in backticks.
- If a project has been archived, do not add new live events for that project here.

## Entry Format
Use this header:

`### YYYY-MM-DD [Project: <Name>]`

Example:

### 2026-02-16 [Project: Example Project]
- **Received**: `<Sender> -> <Recipient>` (`email/portal/post/chat`)
  - Files: `<File1>`, `<File2>`
  - Stored at: `operations/Inbox/YYYY-MM-DD_From_<Source>/`
  - Analysis: `operations/Context/Current/FileAnalysisCurrent.md` -> `DOC-00X`
- **Sent**: `<Sender> -> <Recipient>`
  - Subject: `"<subject>"`
  - Stored at: `operations/Outbox/YYYY-MM-DD_<Description>/`
  - Outgoing record: `operations/Context/Current/FileAnalysisCurrent.md` -> `OUT-00X`
- **Decision**: `<what was decided>`
  - Strategy reference: `operations/Context/Current/StrategyCurrent.md`

## Archive Tombstone Format
When running Archive for a project, leave this type of note:

### YYYY-MM-DD [Project: <Name>]
- **[ARCHIVED]** All Current entries for this project moved to:
  - `operations/Context/Archive/ChronologyArchive.md`
  - `operations/Context/Archive/StrategyArchive.md`
  - `operations/Context/Archive/FileAnalysisArchive.md`
- Trigger: `Archive` command
- Note: Business-general items remain in Current files.

## Current Entries
<!-- Add new dated entries below this line -->
