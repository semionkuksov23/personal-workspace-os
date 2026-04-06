# workspace-os

A structured operating system for managing business operations with Claude Code. Drop it into any project and get instant access to agentic commands, skill-based workflows, and a battle-tested filing system.

## What is this?

`workspace-os` is a workspace scaffold that turns Claude Code into a full-stack business operations assistant. It provides:

- **15 built-in commands** for common workflows (setup, project management, email, transcription, OCR, reporting, planning)
- **7 pluggable skills** that activate on-demand based on what you're doing
- **A structured ops folder** with deterministic filing rules, audit trails, and lifecycle governance
- **Context files** that give the agent persistent memory across sessions

It's designed for professionals who manage multiple concurrent projects and need strict auditability without sacrificing speed.

## Quick start

```bash
# Clone into your project
git clone https://github.com/semionkuksov23/workspace-os.git my-workspace
cd my-workspace

# Open with Claude Code
claude
```

Run the `Setup` command first — it interviews you about your business, populates your context files, and self-deletes when done. After that, use `Prime` at the start of each session to load your business context.

## Architecture

```
workspace-os/
  backend/
    Commands/       # 15 named commands (Setup, Prime, Sweep, Report, etc.)
    Skills/         # 7 skill files (OCR, email, transcription, etc.)
    Scripts/        # Custom automation scripts
  operations/
    Context/
      Current/      # Live context: General, Chronology, Strategy, FileAnalysis
      Archive/      # Completed project data
    Inbox/          # Incoming files, organized by date + source
    Outbox/         # Confirmed sent items
    Drafts/         # Work-in-progress (never auto-sent)
    Records/        # Contracts, evidence, permanent files
    Research/       # Policy docs, research outputs
    Operations/
      OCR/          # OCR text outputs
      Transcripts/  # Audio/video transcription outputs
    Admin/          # Contacts, invoices, fee documents
```

## Commands

| Command | What it does |
|---------|-------------|
| `Archive` | Move completed project data from Current to Archive |
| `Create-Plan` | Design a step-by-step implementation plan |
| `EmailDraft` | Draft outgoing correspondence (.docx) |
| `EmailRead` | Read and process incoming email |
| `EmailSent` | File confirmed-sent email to Outbox |
| `Implement` | Execute a confirmed plan |
| `NewProject` | Register a new project — populates all context files interactively |
| `PermitAll` | Enable full-permission mode for batch operations |
| `Prime` | Load business context and select a project to work on |
| `Report` | Generate project or business-wide reports for a date range |
| `Screen` | Screen and classify incoming documents |
| **`Setup`** | **Start here** — one-time workspace setup that interviews you about your business, then self-deletes |
| `Sweep` | Scan for unfiled items and route them to the right folders |
| `Transcribe` | Audio/video transcription via faster-whisper (CUDA) |
| `US` | UltraSearch across large datasets |

## Email setup

The email commands (`EmailRead`, `EmailDraft`, `EmailSent`) operate via an MCP (Model Context Protocol) server that connects to your email provider. To set it up, simply ask your agent — it will walk you through the configuration. We recommend creating a dedicated folder (e.g. `mcp-setup/`) outside the workspace to keep MCP credentials and config separate from your project files.

## Skills

Skills activate automatically based on what you're doing:

| Skill | Triggers when you... |
|-------|---------------------|
| **accessemail** | Check, read, or process email |
| **draft-governance** | Create, review, or send any draft |
| **llm-enhanced-ocr** | Process scanned PDFs or correct OCR output |
| **safety-guard** | Delete, move, or overwrite files in protected folders |
| **video-analysis** | Analyze or inspect YouTube videos |
| **web-research** | Research or capture information from websites |
| **workspace-skill** | Always active -- core behavioral spec |

## Context system

The workspace maintains four living context files that persist across sessions:

- **General.md** -- Active projects register, business identity, current priorities
- **ChronologyCurrent.md** -- Dated event trail across all projects
- **StrategyCurrent.md** -- Strategy snapshots, risks, decision rationale
- **FileAnalysisCurrent.md** -- Document index with IDs, paths, and status

When a project completes, `Archive` moves its entries to the archive files while preserving all IDs and cross-references.

## Conventions

- **ISO dates** everywhere (`YYYY-MM-DD`)
- **Stable IDs**: `DOC-###` (incoming), `EMAIL-###` (messages), `OUT-###` (outgoing), `ISSUE-###` (gaps), `TASK-###` (actions)
- **No-duplicate filing**: move, don't copy (unless explicitly requested)
- **Draft safety**: nothing leaves `Drafts/` until you confirm it was sent

## Customization

1. Run `Setup` to configure your workspace (recommended for first-time users)
2. Use `NewProject` to register projects — or edit `operations/Context/Current/General.md` directly
3. Add or modify commands in `backend/Commands/`
4. Add domain-specific skills in `backend/Skills/`
5. Drop incoming files in `operations/Inbox/` and run `Sweep`

## License

MIT
