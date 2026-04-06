# Skill: Draft Governance

## Purpose

Enforce explicit lifecycle states on outgoing documents and protect sent items from accidental modification.

## When This Skill Applies

- Creating, editing, reviewing, or sending any draft in `operations/Drafts/`
- Any operation on files in `operations/Outbox/`
- The user says "send this," "this is sent," "mark as sent," or similar
- The user asks to edit or modify a previously sent document
- The `Sweep` command encounters draft or outbox files

## Relationship to workspace-skill.md

This skill **extends** the "Outgoing Governance: Drafts -> Outbox" section.
The existing rules (keep in Drafts until confirmed sent, never assume sent, update context files after send) remain fully in effect.
This skill adds lifecycle state tracking, version preservation, and edit-after-send protections on top.

## Document Lifecycle States

### Drafting

- File is in `operations/Drafts/`
- Freely editable by the agent
- No confirmation needed for changes
- This is the default state for any new outgoing document

### Under Review

- File is in `operations/Drafts/` but the user has indicated it is being reviewed
- Triggered when the user says "review this," "check this," "look this over," or similar
- The agent may suggest changes (corrections, improvements) but must not apply them without confirmation
- Present suggestions as a list; wait for the user to approve each or say "apply all"

### Sent / Locked

- File has moved to `operations/Outbox/<YYYY-MM-DD>_<Description>/`
- The agent must not modify this file under any circumstances
- If the user asks to edit a sent file, follow the Edit-After-Send Protocol below

## State Tracking

Track document state in the corresponding `FileAnalysisCurrent.md` entry using a `Status` field:

```
Status: Drafting | Under Review | Sent | Locked
```

Update the Status field whenever the document transitions between states.
Add a chronology entry for each state transition.

## Draft Versioning Protocol

When a draft undergoes a substantial rewrite (not minor typo fixes):

1. Before overwriting, save the current version as:
   - `<filename>_prev_<YYYY-MM-DD>.<ext>`
   - Place it in the same directory as the current draft
2. Overwrite the main draft file with the new version
3. Note the version save in the FileAnalysis entry

What counts as "substantial":
- Rewriting more than one-third of the document
- Changing the structure (adding/removing sections)
- Changing the recipient, subject, or key ask
- The user explicitly says "rewrite this" or "start over"

What does NOT count (no version save needed):
- Fixing typos or grammar
- Minor wording tweaks
- Adding a sentence or two

## Edit-After-Send Protocol

When the user asks to edit a document that is in Sent/Locked state:

1. **Do not modify the original** in Outbox.
2. **Inform the user**: "This document is in Outbox and marked as sent. I will create a new draft based on it."
3. **Create a new draft** in `operations/Drafts/`:
   - Filename: `v2_<original_filename>` (or `v3_`, `v4_` for subsequent revisions)
   - Copy the content from the sent original
   - Apply the requested changes to the new draft
4. **Create a new FileAnalysis entry** for the new draft:
   - Use a new `OUT-###` ID
   - Reference the original sent document ID in the entry
   - Set Status to `Drafting`
5. **Add a chronology entry** noting the revision was created from a sent original.

## Transition Rules

| From | To | Trigger |
|---|---|---|
| Drafting | Under Review | User says "review" / "check" / "look over" |
| Under Review | Drafting | User says "make changes" / "update it" / "go back to editing" |
| Drafting | Sent/Locked | User confirms sent + files moved to Outbox |
| Under Review | Sent/Locked | User confirms sent + files moved to Outbox |
| Sent/Locked | (new draft) | User asks to edit -- triggers Edit-After-Send Protocol |

## Integration with Safety Guard

If the `safety-guard.md` skill is active:
- Any attempt to modify a Sent/Locked file is classified as a **Red operation**
- The Edit-After-Send Protocol satisfies the safety requirement by creating a copy instead of modifying the original

## Output

- Document state tracked in FileAnalysis entries
- Previous versions preserved with `_prev_` naming
- Sent documents protected from modification
- New drafts created from sent originals when edits are needed
- Chronology entries for all state transitions
