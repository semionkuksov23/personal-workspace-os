# Command: EmailRead

## Purpose
Process incoming email from Gmail into the workspace's filing system with deduplication, attachment handling, and context file updates.

## Gmail Account
All email operations in this workspace use: `semionkuksov23@gmail.com`
When calling any gmail MCP tool, always pass `account="semionkuksov23@gmail.com"`.

## Workspace Type Detection

Before starting, detect workspace type:
- If `Cases/` directory exists at workspace root → **Type 3** (multi-case litigation workspace)
  - If multiple cases exist under `Cases/`, ask user which case to file into
- If `operations/` directory exists at workspace root → **Type 2** (single-project workspace)

## Inputs
- Optional search query (sender, subject, date range)
- Optional project tag(s) (Type 2) or case selection (Type 3)

## Steps

### Step 1 — Search & Display

Default call: `gmail_recent(max_results=10, label_filter="INBOX")`. Only use `gmail_search` if the user specifies a sender, subject, date range, or other filter.

Group the fetched emails into threads. Display a summary table:
```
Found 10 emails across 6 threads:

| #  | Thread Subject                            | From                | Last Email        | Msgs | Attachments |
|----|------------------------------------------|---------------------|-------------------|------|-------------|
| 1  | Re: Правка 5 — Устав и Условия           | Касаткин Л.К.       | 2026-03-04 11:42  | 3    | 2 (docx)    |
| 2  | Holiday Inn Corby — Q1 VAT return         | Andrew Brown         | 2026-03-04 09:15  | 1    | 1 (pdf)     |
| 3  | Re: Villasol board resolution             | Tatiana Hilton       | 2026-03-03 17:30  | 5    | —           |
| 4  | Invoice #6370460                          | HMCTS                | 2026-03-03 14:08  | 1    | 1 (pdf)     |
| 5  | Re: Planning application update           | Amjid Jabbar         | 2026-03-02 16:55  | 2    | —           |
| 6  | Fwd: Insurance renewal docs               | Stokoe Partnership   | 2026-03-01 10:22  | 1    | 3 (pdf)     |

Which threads to process? (numbers, comma-separated, or "all"):
```

Key points:
- Header line states total emails fetched and thread count
- "Last Email" shows `YYYY-MM-DD HH:MM` (not date-only)
- "Attachments" column shows count and dominant file type(s) at a glance so the user can decide what's worth processing

Ask user which threads to process (by number, comma-separated list, or "all").

### Step 2 — Deduplication

For each selected thread:
1. Search workspace for existing transcript files containing the same Gmail thread ID (`GmailThreadId:`).
2. Identify which message IDs within the thread are already filed (match against `GmailMessageId:` in existing transcripts).
3. Only process emails with message IDs NOT already present in existing transcripts.
4. For attachments: skip download if a file with matching filename AND size already exists in the workspace filing location.
5. Report deduplication results: "Thread has N messages, M already filed, processing K new."

If all messages in a thread are already filed, skip it and notify the user.

### Step 3 — Create Filing Pack

Determine the filing destination based on workspace type:

- **Type 2**: `operations/Inbox/YYYY-MM-DD_From_<SenderName>/`
- **Type 3**: Ask which case (if not already determined) → `Cases/<CaseName>/Received/YYYY-MM-DD_From_<SenderName>/`

Use the date of the most recent email in the thread for `YYYY-MM-DD`.
Use the sender's display name (last name or organisation) for `<SenderName>`, sanitised for filesystem safety.

Create the directory if it does not exist.

### Step 4 — Process Each Email in Thread

For each new (non-duplicate) email in chronological order:

1. **Read full email** with `gmail_read(message_id=...)`.
2. **Create transcript file** named `<YYYY-MM-DD>_<SenderLastName>_<short_subject>_transcript.txt` with this metadata header:
   ```
   EmailTranscript
   GmailThreadId: <thread_id>
   GmailMessageId: <message_id>
   From: <sender>
   To: <recipients>
   CC: <cc>
   Date: <date>
   Subject: <subject>

   <body text>
   ```
3. **List attachments** with `gmail_list_attachments(message_id=...)`.
4. **Filter attachments**:
   - **DELETE/SKIP**: Images < 10KB (logos/signatures), files named `image001.*`, `image002.*`, `logo.*`, `signature.*`
   - **KEEP & DOWNLOAD**: PDFs, DOCX, XLSX, EML, TXT, CSVs, images > 10KB, and any other substantive file types
5. **Download kept attachments** to the filing pack directory using `gmail_download_attachment`.
6. **Post-download dedup**: After downloading each attachment, scan the workspace's filing directories for any existing file with the same name and size. If a duplicate is found:
   - Delete the freshly downloaded copy
   - Log: "Skipped `<filename>` — duplicate of existing file at `<path>`"
   This catches attachments forwarded across different threads.
7. **OCR check for PDFs**: For each downloaded PDF, check if it is scanned (no text layer) using PyMuPDF (`fitz`) page text extraction. If all pages are whitespace-only, run OCR using the llm-enhanced-ocr skill and save output to `operations/Operations/OCR/<filename>_OCR.txt` (Type 2) or the equivalent workspace OCR path.

### Step 5 — Assign Document IDs

Assign stable IDs following workspace conventions:
- **Type 2**: Each email gets `EMAIL-###` (next available number). Each significant attachment gets `DOC-###`.
- **Type 3**: Each email gets `EMAIL-XX-###` where `XX` is the case prefix. Each significant attachment gets `DOC-XX-###`.

Determine next available IDs by scanning existing context files.

Update workspace context files:
- **ChronologyCurrent.md** — Add dated entry for each email received, e.g.:
  ```
  ### YYYY-MM-DD
  - [EMAIL-###] Email received from <Sender>: "<Subject>". Filed at `<path>`. [Project: <Name>]
  - [DOC-###] Attachment "<filename>" from EMAIL-###. Filed at `<path>`. [Project: <Name>]
  ```
- **FileAnalysisCurrent.md** — Add document analysis entry for each significant attachment:
  ```
  ## DOC-### — <filename>
  - **Source**: EMAIL-### from <Sender>, <date>
  - **Type**: <PDF/DOCX/XLSX/etc.>
  - **Filed at**: `<path>`
  - **Summary**: <brief description of content>
  - **OCR**: <path to OCR output if applicable, or "N/A — has text layer">
  ```
- **StrategyCurrent.md** — If an email implies a required action, deadline, or open question, add an entry to the Action Items or Risk Register section:
  ```
  - [EMAIL-###] <action description> (from <Sender>, <date>)
  ```
  If no actionable items are found, do not add anything — only write when there is genuine follow-up.

### Step 6 — Completion Summary

Report what was processed:
```
Processed N threads (X new emails filed, Y attachments downloaded, Z PDFs OCR'd).
Skipped M emails (already filed). Skipped P attachments (post-download duplicates).
Context files updated: ChronologyCurrent.md, FileAnalysisCurrent.md, StrategyCurrent.md.
```

Then show a **"Thread relevance"** block listing each filed email with a one-sentence explanation of how it relates to the workspace's active project(s), what info/action it provides, and whether follow-up is needed. Draw on the email content read in Step 4 and the project register in General.md.

```
Thread relevance:
- [EMAIL-###] <Subject> from <Sender> — <one-sentence relevance to workspace project, action provided, follow-up needed or not>
- [EMAIL-###] ...

Would you like me to draft a reply to any of these threads?
```

## Output
- Filing pack paths created
- Transcript files written
- Attachments downloaded
- Document IDs assigned
- Context files updated
