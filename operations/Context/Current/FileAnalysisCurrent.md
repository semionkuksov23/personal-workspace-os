# FileAnalysisCurrent

## Rules
- This file stores document analysis for active projects only.
- Every entry must contain one or more project tags.
- If information is not found here, check `operations/Context/Archive/FileAnalysisArchive.md`.
- Every entry must use stable IDs and exact paths in backticks.

## ID Convention
- Incoming docs: `DOC-###`
- Emails/messages: `EMAIL-###`
- Outgoing items: `OUT-###`
- Issues/gaps: `ISSUE-###`
- Tasks/actions: `ACT-###`

## Required Entry Template (Documents)
### DOC-001: <Short Title> [Project: <Name>]
| Field | Value |
|---|---|
| Project Tags | `[Project: <Name>]` |
| File | `<filename.ext>` |
| Location | `operations/Inbox/...` or `operations/Records/...` |
| Extracted Text (if any) | `operations/Operations/OCR/<filename>_OCR.txt` |
| Date | `<YYYY-MM-DD or Unknown>` |
| Type | `<Contract / Email / Policy / Invoice / Other>` |
| Source | `<Person/Org>` |
| Status | `<Unreviewed / Reviewed / Needs Clarification>` |

#### Key Facts (5-15 bullets minimum)
- `<Fact>`

#### Relevance
- `<Why this matters to the tagged project(s)>`

#### Constraints / Risks / Gaps
- `<Constraint or uncertainty>`

#### Follow-ups
- `<Action>`

## Required Entry Template (Outgoing)
### OUT-001: <Short Title> [Project: <Name>]
| Field | Value |
|---|---|
| Project Tags | `[Project: <Name>]` |
| File | `<filename.ext>` |
| Location | `operations/Drafts/` or `operations/Outbox/YYYY-MM-DD_<Description>/` |
| Date Prepared | `<YYYY-MM-DD>` |
| Date Sent | `<YYYY-MM-DD or NOT YET SENT>` |
| Type | `<Email / Letter / Submission / Other>` |
| Status | `<Draft / Sent - Awaiting Response / Complete>` |

#### Recipients
- `<Name> (<Role>)`

#### Purpose
- `<One paragraph>`

#### Content Summary
- `<Bullet>`

#### Next Steps
- `<Action>`

#### Cross References
- `operations/Context/Current/StrategyCurrent.md`
- `operations/Context/Current/ChronologyCurrent.md`

## Special Handling Markers
- OCR pending/complete path: `operations/Operations/OCR/...`
- Missing attachment placeholder status: `Missing - Re-forward required`
- Multi-project relevance is allowed; keep all project tags in one entry.

## Current Entries
<!-- Add new entries below this line -->
