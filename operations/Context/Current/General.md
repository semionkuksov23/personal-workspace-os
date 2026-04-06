# General Business Context

## Workspace Identity
- Business Name: `<REQUIRED>`
- Workspace Root: `Template/`
- Last Updated: `<YYYY-MM-DD>`
- Maintained By: `<Name>`

## Purpose
This workspace manages multiple concurrent projects for one business.
Use the Current files for active work and Archive files for completed-project history.

If information is not found in Current files, check:
- `operations/Context/Archive/ChronologyArchive.md`
- `operations/Context/Archive/StrategyArchive.md`
- `operations/Context/Archive/FileAnalysisArchive.md`

## Parties and Stakeholders
| Name | Role | Organization | Contact | Notes |
|---|---|---|---|---|
| `<Person>` | `<Role>` | `<Org>` | `<Email/Phone>` | `<Notes>` |

## Active Projects Register
| Project Name | Status | Owner | Started | Current Priority | Key Paths |
|---|---|---|---|---|---|
| `<Project-Name>` | `Active` | `<Owner>` | `<YYYY-MM-DD>` | `<High/Med/Low>` | `operations/Inbox/...`, `operations/Records/...` |

## Current Tasks Register
| Task ID | Project(s) | Summary | Owner | Due Date | Status |
|---|---|---|---|---|---|
| `TASK-001` | `[Project: <Name>]` | `<Task summary>` | `<Owner>` | `<YYYY-MM-DD>` | `Open` |

## Recently Completed Projects
| Project Name | Completion Date | Archive References | Notes |
|---|---|---|---|
| `<Project-Name>` | `<YYYY-MM-DD>` | `operations/Context/Archive/*` | `<Reason/Outcome>` |

## Business-Wide (Non-Archivable) Knowledge
- Items here are business-general and must stay in Current context.
- Examples:
  - Legal entity profile
  - Risk appetite / governance baseline
  - Core operating procedures
  - Preferred naming standards

## Information Location Map
- Current event log: `operations/Context/Current/ChronologyCurrent.md`
- Current planning and actions: `operations/Context/Current/StrategyCurrent.md`
- Current document meaning and evidence value: `operations/Context/Current/FileAnalysisCurrent.md`
- Archive event log: `operations/Context/Archive/ChronologyArchive.md`
- Archive strategy history: `operations/Context/Archive/StrategyArchive.md`
- Archive document analysis: `operations/Context/Archive/FileAnalysisArchive.md`

## Change Log
- `<YYYY-MM-DD>`: Initialized template workspace and baseline context files.
