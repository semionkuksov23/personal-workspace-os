# Command: Setup

## Purpose
One-time guided workspace setup. Interview the user about their business, populate baseline context files, then self-delete.

## Trigger
- First time opening the workspace after cloning.
- User runs `Setup` explicitly.

## Steps

### Phase 1 — Business Discovery

Ask the following conversationally, one or two questions at a time. Do not present a form.

1. **Business type**: "What kind of business or operation is this workspace for? For example: a limited company, sole trader, self-employed professional, partnership, charity, personal use, etc."
2. **Business name**: "What is the name of the business or operation?"
3. **Business description**: "In a sentence or two, what does the business do? For example: 'We manage residential construction projects in the Midlands' or 'I run a freelance software consultancy.'"
4. **Maintainer**: "Who will be the primary person maintaining this workspace? (Name and contact email/phone if available.)"
5. **Key stakeholders**: "Who are the key people involved in the business? Think owners, directors, employees, advisors, accountants, solicitors — anyone the workspace should know about. For each person, give me their name, role, and organisation if applicable."
   - Ask follow-up questions until the user is satisfied the list is complete.
6. **Governance baseline**: "Any core operating procedures or policies the workspace should know about? For example: risk appetite, compliance requirements, approval chains, legal entity details. Say 'skip' if none for now."
7. **File naming convention**: "How would you like files to be named across projects? Common options:
   - `YYYY-MM-DD_Description` (date-first, good for chronological sorting)
   - `ProjectCode_Description` (project-first, good for grouping)
   - `YYYY-MM-DD_ProjectCode_Description` (both)
   - Or describe your own preference."
8. **Workspace phase**: "What phase best describes the business right now? Operations (steady-state), Startup (early stage), Scaling (growing), or Transition (restructuring/winding down)?"

### Phase 2 — Populate Context Files

Using the collected answers:

1. **Update `operations/Context/Current/General.md`**:
   - Fill in Workspace Identity (Business Name, Maintained By, Last Updated = today)
   - Fill in Purpose paragraph with business description
   - Populate Parties and Stakeholders table with all named people
   - Add file naming convention and governance notes under Business-Wide Knowledge
   - Add Change Log entry: `YYYY-MM-DD: Initial workspace setup completed.`

2. **Update `operations/Context/Current/StrategyCurrent.md`**:
   - Set Workspace Phase
   - Add any governance/policy items as Business-General Strategic Decisions
   - Set Last Updated = today

### Phase 3 — First Project Offer

Ask: "Would you like to set up your first project now, or do that later with the `NewProject` command?"

- If **yes**: run the full `NewProject` command flow inline (read and execute `backend/Commands/NewProject.md`).
- If **later**: skip to Phase 4.

### Phase 4 — Self-Deletion

1. Read `backend/Skills/workspace-skill.md`.
2. Remove the line `- \`Setup\` -> \`backend/Commands/Setup.md\`` from the Command Dispatch Rule section.
3. Delete this file (`backend/Commands/Setup.md`).
4. Report: **"Setup complete. The workspace is now configured. The Setup command has been removed — use `NewProject` to add projects going forward."**

## Output
- Fully populated General.md and StrategyCurrent.md
- Optional: first project registered (if user chose to set one up)
- Setup command removed from workspace
