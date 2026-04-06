# Command: EmailDraft

## Purpose
Compose an outgoing email draft, save it locally as `.docx`, and update context files. Does NOT send or create a Gmail draft unless the user explicitly requests it.

## Gmail Account
All email operations in this workspace use: `semionkuksov23@gmail.com`
When calling any gmail MCP tool, always pass `account="semionkuksov23@gmail.com"`.

## Workspace Type Detection

Before starting, detect workspace type:
- If `Cases/` directory exists at workspace root → **Type 3** (multi-case litigation workspace)
  - Ask user which case this draft relates to
- If `operations/` directory exists at workspace root → **Type 2** (single-project workspace)

## Inputs
- Recipient(s)
- Subject
- Key points or instructions for content
- Optional: reference to prior correspondence or document IDs
- Optional: project tag(s) (Type 2) or case selection (Type 3)

## Steps

### Step 1 — Gather Requirements

Ask user for:
1. Recipient name and email address
2. Subject line (or topic for the agent to formulate one)
3. Key points to cover in the email
4. Tone/register (formal, semi-formal, brief)
5. Any documents to reference or attach references to

If Type 3: confirm which case this relates to (if not already clear from context).

If any required information is missing, ask one clarifying question before proceeding.

### Step 2 — Research Context

Before composing, gather relevant context:
1. Read `ChronologyCurrent.md` — identify recent events relevant to this correspondence.
2. Read `FileAnalysisCurrent.md` — identify documents the email may need to reference.
3. Read `StrategyCurrent.md` — check for strategic considerations or constraints.
4. Search for prior correspondence with the recipient:
   - Search transcript files in `operations/Inbox/` (Type 2) or `Cases/<CaseName>/Received/` (Type 3) for the recipient's name.
   - Search `operations/Outbox/` (Type 2) or `Cases/<CaseName>/Sent/` (Type 3) for previous outgoing emails.
5. Identify relevant document IDs (DOC-###, EMAIL-###) to reference in the email body.

### Step 3 — Compose Draft

Write the email text with:
- Professional salutation appropriate to the relationship and context
- Structured body covering all user-specified key points
- References to relevant documents using plain-language descriptions (dates, titles, parties) — NOT internal IDs
- Clear closing with any requested actions or next steps
- Professional sign-off
- **No internal references in email text**: Do NOT include internal tracking IDs (e.g. `OUT-###`, `DOC-###`, `EMAIL-###`, case-prefixed IDs like `CW-OUT-23`) in the email body. These are internal-only and meaningless to the recipient. Use plain-language descriptions instead (e.g. "the letter dated 5 March" rather than "[DOC-14]").

### Step 4 — Save as .docx

Determine the filing destination:
- **Type 2**: `operations/Drafts/YYYY-MM-DD_Draft_<Description>.docx`
- **Type 3**: `Cases/<CaseName>/Drafts/YYYY-MM-DD_Draft_<Description>.docx`

Create a Python script using `python-docx` that generates the `.docx` with:
- **Header block** containing metadata:
  - To: `<recipient>`
  - From: `<user/workspace identity>`
  - Date: `<current date>`
  - Subject: `<subject>`
  - Reference IDs: `<relevant DOC/EMAIL IDs>`
- **Body text** with proper paragraph formatting
- Save to the determined path

Run the script to generate the `.docx` file.

Ensure the `Drafts/` directory exists before saving. Create it if missing.

### Step 5 — Assign OUT-### ID

Assign the next available outgoing document ID:
- **Type 2**: `OUT-###` (next available number)
- **Type 3**: `OUT-XX-###` where `XX` is the case prefix

Determine next available ID by scanning existing context files.

Update context files:
- **ChronologyCurrent.md** — Add entry:
  ```
  ### YYYY-MM-DD
  - [OUT-###] Draft prepared: email to <Recipient> re "<Subject>". Saved at `<path>`. Status: Drafting. [Project: <Name>]
  ```
- **StrategyCurrent.md** — Update if the draft is strategically relevant (e.g., responds to a deadline, advances a negotiation, or addresses an identified risk).

### Step 6 — Present for Review

Display the full draft text inline for user review.

**HARD RULE**: Save locally as `.docx` ONLY. Do NOT create a Gmail draft via `gmail_create_draft` unless the user explicitly asks for it (e.g., "put it in Gmail drafts", "create a Gmail draft", "save to Gmail").

Ask the user:
1. **Approve as-is** — draft is final, remains in Drafts/ until confirmed sent
2. **Request changes** — specify what to revise, then regenerate the `.docx`
3. **Create Gmail draft** — only if explicitly requested, use `gmail_create_draft` to also save it in Gmail
4. **Discard** — delete the `.docx` and remove context file entries

## Output
- Draft `.docx` path
- OUT-### ID assigned
- Context files updated
- Full draft text displayed for review
