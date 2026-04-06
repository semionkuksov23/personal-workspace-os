# Command: Prime

## Purpose
Prime the agent with current business context at start of session.

## Inputs
- Optional focus project name (may be provided upfront or selected interactively).

## Steps
1. Read:
   - `operations/Context/Current/General.md`
   - `operations/Context/Current/ChronologyCurrent.md`
   - `operations/Context/Current/StrategyCurrent.md`
   - `operations/Context/Current/FileAnalysisCurrent.md`
2. Build a concise startup summary:
   - business identity
   - active projects
   - active tasks
   - high-priority risks
3. List every active project from `General.md`'s "Active Projects Register" (name + current priority).
4. Ask: "Which project do you have an update for or want to continue working on?" Offer "None — general workspace tasks" as an option. If a focus project was already provided as input, skip this prompt and use that project.
5. If user selects a project, load and present a focused summary for that project:
   - project strategy snapshot from `StrategyCurrent.md`
   - pending action items
   - open risks
   - recent chronology entries for that project

## Output Template
- Business: `<Name>`
- Active Projects: `<Count and names>`
- Top Priorities: `<3-5 bullets>`
- Risks: `<3 bullets>`
- Data health: `<Any filing/transcript/OCR gaps>`

### Active Projects
| # | Project | Priority |
|---|---------|----------|
| 1 | `<Project-Name>` | `<High/Med/Low>` |
| 2 | `<Project-Name>` | `<High/Med/Low>` |

**Which project do you have an update for or want to continue working on?** (or "None" for general workspace tasks)
