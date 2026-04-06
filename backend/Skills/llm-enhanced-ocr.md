# Skill: LLM-Enhanced OCR

## Purpose

Upgrade raw Tesseract OCR output by running an LLM correction pass that fixes errors, restores formatting, and produces clean readable markdown.

## When This Skill Applies

- A scanned PDF has been processed by the OCR pipeline defined in `workspace-skill.md`
- The agent is about to save OCR output to `operations/Operations/OCR/`
- The user asks to "clean up," "correct," or "improve" existing OCR text
- The `Sweep` command detects a new scanned PDF and triggers OCR

## Relationship to workspace-skill.md

This skill **extends** the "OCR Pipeline for Scanned PDFs" section.
It does not replace any existing steps. The Tesseract extraction runs first exactly as defined in workspace-skill.md.
This skill adds a post-processing phase after step 3 (saving raw OCR output).

## Protocol

### Step 1: Preserve Raw Output

After Tesseract produces raw OCR text, save it as:
- `operations/Operations/OCR/<original_filename>_OCR_raw.txt`

This file is the unmodified Tesseract output and must never be overwritten or deleted.

### Step 2: Chunk the Raw Text

Split the raw OCR text into chunks of approximately 2000-3000 words each.
- Split at paragraph or page boundaries, not mid-sentence.
- Include a small overlap (1-2 sentences) between chunks to maintain context across boundaries.

### Step 3: LLM Correction Pass

For each chunk, instruct the LLM to:
1. Fix obvious OCR errors (garbled characters, broken words, missing spaces, merged lines).
2. Restore punctuation and capitalisation where clearly wrong.
3. Preserve the original meaning exactly -- do not rephrase, summarise, or add content.
4. Maintain the original document structure (headings, lists, tables, paragraphs).
5. Mark any passage that is too damaged to correct confidently with `[OCR UNCLEAR: ...]`.
6. For non-Latin scripts (e.g. Cyrillic), pay special attention to character substitutions (e.g. Latin "c" for Cyrillic "с").

### Step 4: Reassemble and Save Corrected Output

Reassemble the corrected chunks into a single file:
- `operations/Operations/OCR/<original_filename>_OCR.txt`

Remove the overlap duplicates during reassembly.
Format the output as clean markdown with:
- Page separators: `---` with `<!-- Page N -->` comments
- Headings preserved from the original document structure
- Tables formatted as markdown tables where recognisable

### Step 5: Assess Quality

After correction, classify the output quality:
- **Clean**: text is fully readable, no `[OCR UNCLEAR]` markers
- **Partial**: text is mostly readable, fewer than 5 unclear passages
- **Manual Review Needed**: more than 5 unclear passages or substantial sections unrecoverable

### Step 6: Update File Analysis

In the `FileAnalysisCurrent.md` entry for this document:
- Set the OCR path to the corrected file (`_OCR.txt`)
- Add a note referencing the raw file (`_OCR_raw.txt`) for audit
- Record the quality classification
- If quality is "Manual Review Needed", add a follow-up action for manual verification

## Output

- Raw OCR file: `operations/Operations/OCR/<filename>_OCR_raw.txt`
- Corrected OCR file: `operations/Operations/OCR/<filename>_OCR.txt`
- Quality classification in FileAnalysis entry
- Chronology entry noting OCR correction was performed
