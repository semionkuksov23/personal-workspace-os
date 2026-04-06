# Command: Screen

## Purpose
Process screenshot/photo written exchanges with transcript and retention handling.

## Inputs
- Medium (`Email/WhatsApp/SMS/Chat/Other`)
- Source
- Project tag(s)
- Screenshot source mode:
  - chat attachment not yet saved
  - file already on disk

## Two-Phase Flow (Chat Attachment, No File Path Yet)

### Phase A
1. Ask project tag(s) and source details if unclear.
2. Create destination pack:
   - `operations/Inbox/YYYY-MM-DD_From_Source/`
3. Create sibling transcript:
   - `<basename>_transcript.txt`
4. Include metadata:
   - CapturedDate, Medium, Source, OriginalFile, Notes
5. Set:
   - `OriginalFile: CHAT_ATTACHMENT_PENDING`
   - `Retention: ScreenshotDeletedAfterTranscription (chat attachment)`
6. Add chronology note that screenshot was captured.

### Phase B
1. Move saved screenshot into pack (provenance step).
2. Update transcript `OriginalFile` to real filename.
3. Add `OriginalFileStatus: DeletedAfterTranscription`.
4. Delete screenshot after transcript update (chat-origin only).
5. Update chronology with final storage/update note.

## On-Disk Screenshot Flow
1. File image to destination pack.
2. Create sibling transcript with required fields.
3. Keep image file after filing.
4. Update chronology and file analysis.

## Output
- Paths created/updated
- transcript status
- retention action taken
