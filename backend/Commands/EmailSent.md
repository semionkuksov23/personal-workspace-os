# Command: EmailSent

## Purpose
Confirm and file a sent email by retrieving it from Gmail, comparing it against the local draft, and moving it to the outbox/sent filing location with context file updates.

## Gmail Account
All email operations in this workspace use: `semionkuksov23@gmail.com`
When calling any gmail MCP tool, always pass `account="semionkuksov23@gmail.com"`.

## Workspace Type Detection

Before starting, detect workspace type:
- If `Cases/` directory exists at workspace root → **Type 3** (multi-case litigation workspace)
  - If multiple cases exist under `Cases/`, ask user which case this relates to
- If `operations/` directory exists at workspace root → **Type 2** (single-project workspace)

## Inputs
- Identification of which email was sent: subject line, date, recipient, or reference to a draft OUT-### ID
- Optional: project tag(s) (Type 2) or case selection (Type 3)

## Steps

### Step 1 — Identify What Was Sent

Ask user which email was sent. Accept any of:
- Subject line or keyword
- Date sent
- Recipient name
- Draft reference `OUT-###` ID

Search Gmail sent folder using `gmail_search`:
```
gmail_search(query="in:sent from:me subject:<subject> after:<date>")
```

If multiple results match, display a summary table and ask user to confirm which one:
```
| #  | Subject                          | To                | Date Sent    |
|----|----------------------------------|-------------------|-------------|
| 1  | Re: Application to vary bail     | simi@starck.co.uk | 2026-03-03  |
| 2  | Follow-up on lease terms         | martyn@jones.com  | 2026-03-02  |
```

### Step 2 — Read Sent Email and Compare to Draft

1. Use `gmail_read(message_id=...)` to get the full sent email content.
2. If a local draft exists (identified by OUT-### ID):
   - Locate the draft `.docx` in:
     - Type 2: `operations/Drafts/`
     - Type 3: `Cases/<CaseName>/Drafts/`
   - Read the draft content
   - Compare the sent email against the draft
   - Note any differences (additions, deletions, wording changes)
   - Report differences to the user:
     ```
     Draft vs Sent comparison:
     - Draft had: "We propose a meeting on Tuesday"
     - Sent version: "We propose a meeting on Wednesday"
     ```
3. If no local draft exists, proceed with filing the sent email directly.

### Step 3 — Create Sent Transcript

Create a `.txt` transcript file named `<YYYY-MM-DD>_Sent_<short_description>_transcript.txt` with this metadata header:
```
SentEmailTranscript
GmailThreadId: <thread_id>
GmailMessageId: <message_id>
From: <user>
To: <recipients>
CC: <cc>
Date: <sent_date>
Subject: <subject>
DraftReference: OUT-###

<sent body text>
```

If no draft reference exists, omit the `DraftReference:` line.

### Step 4 — File the Sent Email

Create the filing pack directory:
- **Type 2**: `operations/Outbox/YYYY-MM-DD_<Description>/`
- **Type 3**: `Cases/<CaseName>/Sent/YYYY-MM-DD_<Description>/`

Use the sent date for `YYYY-MM-DD` and a short description derived from the subject for `<Description>`, sanitised for filesystem safety.

Move the transcript into the filing pack directory.

Handle the draft `.docx` if it exists:
- **Default**: Move the draft `.docx` into the sent filing pack folder (for record-keeping alongside the transcript).
- Delete the now-empty draft subfolder from `operations/Drafts/` or `Cases/<CaseName>/Drafts/` if applicable.
- Check for any remaining empty subfolders in the Drafts directory and remove them.

### Step 5 — Update Context Files

Update the following context files:

- **ChronologyCurrent.md** — Add entry:
  ```
  ### YYYY-MM-DD
  - [OUT-###] Email sent to <Recipients> re "<Subject>". Filed at `<path>`. [Project: <Name>]
  ```
  If no OUT-### was previously assigned (no draft existed), assign the next available OUT-### now.

- **FileAnalysisCurrent.md** — If OUT-### already has an entry:
  - Update status from `Drafting` → `Sent/Locked`
  - Update file path to the new Outbox/Sent location

- **StrategyCurrent.md** — Update if strategically relevant:
  - Deadline met or action taken
  - Risk mitigated by the communication
  - Negotiation advanced or position stated

### Step 6 — Completion Summary

Report:
```
Filed sent email: <Subject>
Sent to: <Recipients>
Date: <sent_date>
Filed at: <path>
Draft reference: OUT-### (status updated to Sent)
Context files updated: ChronologyCurrent.md, FileAnalysisCurrent.md, StrategyCurrent.md
```

If no draft reference existed:
```
Filed sent email: <Subject>
Sent to: <Recipients>
Date: <sent_date>
Filed at: <path>
Assigned: OUT-### (new)
Context files updated: ChronologyCurrent.md, StrategyCurrent.md
```

## Output
- Sent transcript path
- Filing pack path
- Draft disposition (moved to sent pack or N/A)
- OUT-### ID and status
- Context files updated
