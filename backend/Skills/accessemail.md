# Skill: Email Access (Read-Only Gmail)

## Purpose

Provide read-only Gmail access via MCP tools, replacing the screenshot→transcribe workflow with direct API email retrieval, reading, and attachment downloading.

## Gmail Account
All email operations in this workspace use: `semionkuksov23@gmail.com`
When calling any gmail MCP tool, always pass `account="semionkuksov23@gmail.com"`.

## When This Skill Applies

- The user asks to check email, read email, or look at incoming correspondence
- The user asks to process, file, or download email attachments
- The user mentions a "screen" or screenshot that is actually an email
- The user asks to find or search for a specific email thread
- The user asks what emails have arrived from a specific person or about a specific topic

## Relationship to workspace-skill.md

This skill **extends** the intake/filing protocol defined in workspace-skill.md. It replaces the manual screenshot→OCR→transcribe intake path with direct Gmail API access. Filing rules, folder structure, and transcript conventions from workspace-skill.md still apply — this skill only changes *how* the email content is obtained.

## Hard Rule: NEVER Send Email

This server has read + compose access (`gmail.readonly` + `gmail.compose` scopes). It **can create drafts** in the user's Gmail Drafts folder but **cannot send, modify, or delete** any email.

If the user asks to send or reply to an email:
1. Prepare the email content (can reference workspace files for context)
2. Use `gmail_create_draft` to save it to the user's Gmail Drafts folder
3. Also save a copy in the workspace `Drafts/` folder per the standard draft governance protocol
4. Tell the user: "Draft created in your Gmail Drafts folder. Please open Gmail to review and send it."
5. Do NOT attempt to call any send API — no such tool exists.

## Available MCP Tools

### 1. `gmail_search`
Search Gmail using the same query syntax as the Gmail search bar.
```
gmail_search(query="from:martyn subject:lease", max_results=10)
```

**Common Gmail query operators:**
- `from:name` / `to:name` — sender or recipient
- `subject:word` — subject line contains word
- `has:attachment` — only messages with attachments
- `after:YYYY/MM/DD` / `before:YYYY/MM/DD` — date range
- `in:inbox` / `in:sent` / `in:trash` — location
- `is:unread` / `is:starred` — message state
- `filename:pdf` — attachment type
- Combine with spaces (AND) or `OR`: `from:martyn OR from:richard`

### 2. `gmail_read`
Read the full content of a single email by message ID (obtained from search or recent results).
```
gmail_read(message_id="18d5a7b3c2e1f0a9")
```
Returns: all headers, plain-text body (or HTML fallback), and attachment metadata.

### 3. `gmail_list_attachments`
List attachments on an email without downloading them.
```
gmail_list_attachments(message_id="18d5a7b3c2e1f0a9")
```
Returns: filenames, MIME types, sizes, and attachment IDs.

### 4. `gmail_download_attachment`
Download a specific attachment to a local directory.
```
gmail_download_attachment(
    message_id="18d5a7b3c2e1f0a9",
    attachment_id="ANGjdJ...",
    filename="invoice.pdf",
    destination_dir="D:/Cursor/Projects/Legal/Collingwood lawsuit/Received/2026-02-25_From_Tatiana/"
)
```

### 5. `gmail_list_labels`
List all Gmail labels (system and user-created) with their IDs. Useful for understanding label structure.
```
gmail_list_labels()
```

### 6. `gmail_recent`
Get the most recent emails, optionally filtered by label.
```
gmail_recent(max_results=5, label_filter="INBOX")
```

### 7. `gmail_create_draft`
Create a draft email in the user's Gmail Drafts folder. Does NOT send — the user must review and send manually from Gmail.
```
gmail_create_draft(
    to="simi@starckuberoi.co.uk",
    subject="Re: Application to vary bail",
    body="Dear Simi, ...",
    cc="",
    thread_id="19c940a0270cab30",      # optional, for replies
    in_reply_to="<original-message-id>", # optional, for threading
)
```

## Workflow

### Standard Email Intake

1. **Search or browse**: Use `gmail_search` or `gmail_recent` to find the relevant email(s).
2. **Read**: Use `gmail_read` to get the full email content.
3. **Download attachments** (if any): Use `gmail_list_attachments` then `gmail_download_attachment` to save files to the correct workspace folder.
4. **Create transcript**: Write a `_transcript.txt` file following the standard metadata format (see below).
5. **File into workspace**: Place all files in the workspace's `Received/` folder using the standard pack convention: `<YYYY-MM-DD>_From_<Source>/`.
6. **Update records**: Add FileAnalysis and Chronology entries per workspace-skill.md.

### Transcript File Format

After reading an email via `gmail_read`, create a transcript file with this structure:

```
---
CapturedDate: YYYY-MM-DD
Medium: Email
Direction: Inbound | Outbound
Source: <sender name and address>
Recipients: <to, cc>
Subject: <email subject line>
GmailMessageId: <message_id>
GmailThreadId: <thread_id>
AttachmentCount: <N>
---

<full email body text>
```

Save as: `Received/<YYYY-MM-DD>_From_<Source>/<descriptive_name>_transcript.txt`

For example: `Received/2026-02-25_From_Martyn/2026-02-25_Martyn_lease_update_transcript.txt`

### Filing Integration

- Downloaded attachments go into the same `Received/<YYYY-MM-DD>_From_<Source>/` pack folder alongside the transcript
- If the workspace uses an `Inbox/` folder instead of `Received/`, use that
- Follow the workspace's existing filing conventions — this skill changes the *source* of the data, not where it goes

## Limitations

- **No pagination**: Each search returns at most one page of results (max 100). For very broad searches, narrow the query with date ranges or other filters.
- **Body truncation**: Message bodies longer than 50,000 characters are truncated.
- **HTML fallback**: If no plain-text part exists, the HTML body is returned as-is. Consider extracting readable text if it's cluttered.
- **No send/compose/delete**: Read-only access only.
