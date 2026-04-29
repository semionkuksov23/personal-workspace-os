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
- References to relevant documents using plain-language descriptions (dates, titles, parties) — NOT internal IDs (see Step 3a below — this rule is enforced)
- Clear closing with any requested actions or next steps
- Professional sign-off

### Step 3a — HARD RULE: NO INTERNAL REFERENCES IN EMAIL BODY

This rule has failed in practice when stated as a passing bullet. Treat this step as load-bearing — apply it during composition, then verify in Step 4a.

**Why this matters.** The recipient is the counterparty (NCA, solicitor, opposing party, bank). Internal IDs are workspace-only bookkeeping. Including them is at best confusing and at worst leaks the structure of internal records the user does not want disclosed.

**The agent composes from internal context that is itself ID-laden** (Strategy.md, Chronology.md, Case-File-Analysis.md, prior transcripts). Quoting that context directly will drag IDs through unless the agent actively converts every reference to plain language at the moment of writing.

**The forbidden token shapes (non-exhaustive):**

- `OUT-###` / `OUT-XX-###` (e.g. `OUT-SC-012`, `OUT-CW-073`, `OUT-IM-034`)
- `EMAIL-###` / `EMAIL-XX-###` (e.g. `EMAIL-SC-025`, `EMAIL-CW-066`)
- `DOC-###` / `DOC-XX-###` (e.g. `DOC-CW-019`, `DOC-IM-013`)
- `ISSUE-###` / `ISSUE-XX-###`
- `RES-###` / `RES-XX-###`
- `TASK-###` / `TASK-XX-###` / `ACT-###` / `ACT-XX-###`
- `CALL-###` / `CALL-XX-###`
- `EVENT-###` / `EVENT-XX-###`
- `SCREEN-###` / `SCREEN-XX-###`
- Any other prefix-### pattern that originates in workspace context files

**Conversion rule — always swap an internal ID for a plain-language descriptor:**

| Bad (internal) | Good (plain-language) |
|---|---|
| `(OUT-SC-012, acknowledged at EMAIL-SC-025)` | `(my notification of 12 March 2026, acknowledged by you on 13 March 2026)` |
| `as set out in OUT-CW-073` | `as set out in my email of 9 April 2026` |
| `per DOC-CW-019` | `per the BVI agent's certification dated 9 April 2026` |
| `referenced in EMAIL-SC-042` | `as Nationwide stated in its email of 16 April 2026` |
| `see OUT-IM-034` | `see my email to Simi of 10 April 2026` |

The descriptor must give the recipient enough to find the item on their own side: typically date, sender, subject/topic, and (where useful) document reference number used by the counterparty.

**Two edge cases:**

1. **Counterparty's own reference numbers ARE allowed and encouraged.** If Nationwide's complaint reference is `COM0023907`, or the FOS case reference is `PNX-5962278-F9P2`, those are the recipient's own bookkeeping and should be used. The rule only excludes IDs that originate in this workspace.
2. **Court / case numbers ARE allowed.** Case No. `T20230250`, claim numbers, court file references — all external-facing identifiers, all permitted.

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

### Step 4a — Mandatory Pre-Save Scan: No Internal References

Before considering Step 4 complete, scan the composed email body (the text that will be sent — NOT the metadata header) for forbidden token shapes. If ANY match is found, do NOT save. Convert every match to a plain-language descriptor per Step 3a, then re-scan.

**The scan must catch (case-insensitive, word-boundary):**

```
\b(OUT|EMAIL|DOC|ISSUE|RES|TASK|ACT|CALL|EVENT|SCREEN)(-[A-Z]{2})?-\d{2,4}\b
```

In practice, before writing the `.docx`:

1. Inspect the composed body string for matches against the regex above.
2. For each match, replace with the appropriate plain-language descriptor (date + sender + topic + counterparty reference if any). If no plain-language replacement is possible, remove the reference entirely — never leave it in.
3. Re-scan. Only save once the scan returns zero matches.
4. Briefly note in the user-facing summary: "Pre-save scan: clean — N internal-ID matches converted to plain language." (If N=0, just "Pre-save scan: clean.")

**Note on the metadata header.** The header block written into the `.docx` (Step 4 — To/From/Date/Subject/Reference IDs) is allowed to contain internal IDs because it is workspace-only metadata, not part of the sent body. Only the body that will be copied into Gmail is subject to the scan.

**If the user has explicitly asked for an internal ID to remain** (rare — e.g. an internal-only memo masquerading as an email), confirm the override in writing and note it in the user-facing summary. Default is always: scan, convert, save clean.

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
