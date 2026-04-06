# Workspace Auto-Loader

## Scope
Applies to all work inside `Workspaces/Template/` and subfolders.

## Mandatory Read First
Before any work, read:
- `backend/Skills/workspace-skill.md`

## Operating Note
The skill file is authoritative for behavior, command dispatch, filing rules, and lifecycle governance.

## Supplementary Skills
When a trigger condition matches, read the corresponding skill file before proceeding:

- `backend/Skills/llm-enhanced-ocr.md`
  Trigger: processing a scanned PDF, running OCR, or correcting OCR output
- `backend/Skills/safety-guard.md`
  Trigger: any operation that deletes, moves, or overwrites files in Inbox, Outbox, Records, or Admin
- `backend/Skills/web-research.md`
  Trigger: user asks to research, look up, or capture information from a website
- `backend/Skills/draft-governance.md`
  Trigger: creating, reviewing, sending, or editing any draft or outbox item
- `backend/Skills/accessemail.md`
  Trigger: user asks to check email, read email, process incoming email, download email attachments, file email correspondence, or runs EmailRead/EmailDraft/EmailSent command
- `backend/Skills/video-analysis.md`
  Trigger: user asks to analyze, review, describe, watch, or inspect a YouTube video, or provides a YouTube URL for visual analysis
