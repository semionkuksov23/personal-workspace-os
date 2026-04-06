# Command: NewProject

## Purpose
Register a new project in the workspace by interviewing the user and populating all four context files.

## Trigger
- User wants to start tracking a new project.
- User runs `NewProject` explicitly.
- Called inline by the `Setup` command for the first project.

## Preconditions
- Workspace must be configured (General.md has a populated Workspace Identity section).
- Read `operations/Context/Current/General.md` to check existing projects and the file naming convention in Business-Wide Knowledge.

## Steps

### Phase 1 — Project Discovery

Ask the following conversationally, one or two questions at a time:

1. **Project name**: "What is the name of this project?"
2. **Objective**: "What is the goal of this project? In a sentence or two, what are you trying to achieve?"
3. **Owner**: "Who owns or leads this project?" (Suggest names from the existing Stakeholders table if populated.)
4. **Priority**: "What priority level — High, Medium, or Low?"
5. **Key stakeholders**: "Who else is involved in this project? Any new people to add to the workspace contacts?" (Show existing stakeholders for reference.)
6. **Key paths**: "Where will the main files for this project live? Default is `operations/Inbox/`, `operations/Records/`, `operations/Research/` — or specify custom paths."
7. **Initial risks**: "Any known risks or concerns? Say 'none' if not applicable."
8. **Open questions**: "Any open questions or unknowns to track? Say 'none' if not applicable."
9. **First action items**: "Any immediate tasks to register? For each, give a summary, owner, and deadline. Say 'none' to skip."

### Phase 2 — Populate Context Files

Apply the file naming convention from General.md's Business-Wide Knowledge section.

1. **Update `operations/Context/Current/General.md`**:
   - Add row to Active Projects Register: Project Name, Status=`Active`, Owner, Started=today, Priority, Key Paths.
   - If new stakeholders were mentioned, add them to the Parties and Stakeholders table.
   - Add any initial tasks to Current Tasks Register with `TASK-###` IDs and `[Project: <Name>]` tags.
   - Update Last Updated date.

2. **Update `operations/Context/Current/StrategyCurrent.md`**:
   - Add new project strategy section using the template:
     - Project Snapshot (Objective, Owner, Status=Active, Last Reviewed=today)
     - Strategic Decisions table (empty or with initial entries)
     - Action Items (from user's answers)
     - Open Questions (from user's answers)
     - Risk Register (from user's answers, with Impact/Likelihood/Mitigation/Owner)
     - Change Log: `YYYY-MM-DD: Project registered.`

3. **Update `operations/Context/Current/ChronologyCurrent.md`**:
   - Add entry: `### YYYY-MM-DD [Project: <Name>]`
   - Type: **Decision** — "Project registered. Objective: <objective>. Priority: <priority>."

4. **Update `operations/Context/Current/FileAnalysisCurrent.md`**:
   - Only add entries if the user provides initial documents during discovery.
   - Otherwise, skip — the file will be populated as documents arrive.

### Phase 3 — Confirm

Present a summary of everything registered:
- Project name and priority
- Stakeholders involved
- Number of action items and risks logged
- Key paths

Ask: "Does this look correct? Any changes before we finalise?"

Apply corrections if requested, then report: **"Project '<Name>' has been registered. Use `Prime` to load your workspace context."**

## Output
- Updated General.md with new project row and any new stakeholders
- New project strategy section in StrategyCurrent.md
- Initial chronology entry in ChronologyCurrent.md
- Summary confirmation to user
