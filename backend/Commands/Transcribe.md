# Command: Transcribe

## Purpose
Transcribe audio/video recordings using faster-whisper on CUDA with quality profiles, speaker diarization, mandatory monitoring, and automated QC — producing transcripts reliable enough for deep legal, financial, and strategic analysis.

## Inputs
- Audio/video file path (`.mp4`, `.m4a`, `.mp3`, `.wav`, `.ogg`, `.webm`, `.mkv`, `.flac`)
- Language(s) — asked in Step 1
- Description/label — asked in Step 1
- Quality profile — chosen in Step 2

---

## Step 1 — Validate & Ask Language

1. Validate file exists and has a supported extension.
2. Get duration via `ffprobe -v error -show_entries format=duration -of csv=p=0 "<file>"`.
3. **Ask user**: "What language(s) are present in the recording?"
   - Single language → set `language` param (e.g., `"en"`, `"ru"`)
   - Mixed languages → set `language=None` (auto-detect per segment)
4. **Ask user**: Short description/label for file naming (or derive from filename if obvious).

## Step 2 — Comparative Table

Present options with estimated times calculated from actual audio duration. **Table adapts based on language answer.**

### Single Language

| # | Profile | Model | Method | beam | Quality | Est. Time | Notes |
|---|---------|-------|--------|------|---------|-----------|-------|
| 1 | **Light** | large-v3-turbo | Single-pass, VAD on | 1 | Reasonable | ~0.05x duration | Fastest. May miss quiet speech. |
| 2 | **Balanced** | large-v3-turbo | Single-pass, VAD on | 5 | Good | ~0.15x duration | Good for clear recordings. |
| 3 | **Thorough** | large-v3-turbo | 2-pass (base + gap-fill) | 5 | Very Good | ~0.3x duration | Recovers speech in gaps. Recommended for important calls. |
| 4 | **Maximum** | large-v3 | 2-pass + gap-fix (3 passes) | 5 | Excellent | ~0.6x duration | Highest quality. For legal/evidentiary recordings. |

### Multiple Languages (e.g., Russian + English code-switching)

| # | Profile | Model | Method | beam | Quality | Est. Time | Multilingual Notes |
|---|---------|-------|--------|------|---------|-----------|-------------------|
| 1 | **Light** | large-v3-turbo | Single-pass, VAD on, `language=None` | 1 | Reasonable | ~0.08x duration | Auto-detect per segment. May misidentify short switches. |
| 2 | **Balanced** | large-v3-turbo | Single-pass, VAD on, `language=None` | 5 | Good | ~0.2x duration | Better language boundary detection with wider beam. |
| 3 | **Thorough** | large-v3 | 2-pass, `language=None` | 5 | Very Good | ~0.5x duration | large-v3 handles code-switching better than turbo. Gap-fill recovers missed switches. |
| 4 | **Maximum** | large-v3 | 2-pass + gap-fix, `language=None` | 5 | Excellent | ~0.8x duration | Best for multilingual. Gap-fix targets language-boundary gaps. |

**Multilingual handling rules:**
- `language=None` enables per-segment auto-detection (Whisper decides language per ~30s chunk)
- For multilingual at Thorough+ levels, use `large-v3` (NOT turbo) — significantly better code-switching
- Time estimates are higher for multilingual (auto-detect overhead + more language-boundary gaps)
- `.json` output must include `detected_language` field per segment
- QC semantic spot-check must verify language labels are plausible

Show actual estimated minutes based on file duration. Ask user to choose.

## Step 3 — Generate & Run Script

Generate Python script at `backend/Scripts/transcribe_<STEM>.py`.

### Required Boilerplate (all profiles)

```python
import os, sys, json, subprocess, re
sys.stdout.reconfigure(encoding="utf-8")

# cuBLAS DLL fix — MUST come before faster_whisper import
_cublas_dir = os.path.join(
    os.environ["LOCALAPPDATA"],
    r"Python\pythoncore-3.14-64\Lib\site-packages\nvidia\cublas\bin"
)
if os.path.isdir(_cublas_dir):
    os.environ["PATH"] = _cublas_dir + os.pathsep + os.environ["PATH"]
    os.add_dll_directory(_cublas_dir)

from faster_whisper import WhisperModel

def format_ts(seconds):
    h = int(seconds // 3600)
    m = int((seconds % 3600) // 60)
    s = int(seconds % 60)
    ms = int((seconds - int(seconds)) * 1000)
    return f"{h:02d}:{m:02d}:{s:02d},{ms:03d}"

def format_ts_txt(seconds):
    h = int(seconds // 3600)
    m = int((seconds % 3600) // 60)
    s = int(seconds % 60)
    return f"{h:02d}:{m:02d}:{s:02d}"

def seg_to_dict(seg, source="base", lang=None):
    words = []
    if seg.words:
        words = [{"start": w.start, "end": w.end, "word": w.word, "probability": w.probability} for w in seg.words]
    return {
        "start": seg.start,
        "end": seg.end,
        "text": seg.text.strip(),
        "avg_logprob": seg.avg_logprob,
        "no_speech_prob": seg.no_speech_prob,
        "source": source,
        "words": words,
        "detected_language": lang
    }

def get_duration(path):
    r = subprocess.run(
        ["ffprobe", "-v", "error", "-show_entries", "format=duration", "-of", "csv=p=0", path],
        capture_output=True, text=True
    )
    return float(r.stdout.strip())
```

### Profile Configurations

**Single language:**
- **Light**: `large-v3-turbo`, `beam_size=1`, single-pass VAD on, `language=<user_lang>`
- **Balanced**: `large-v3-turbo`, `beam_size=5`, single-pass VAD on, `language=<user_lang>`
- **Thorough**: `large-v3-turbo`, `beam_size=5`, 2-pass (VAD on + gap-fill VAD off for gaps > 1.0s), `language=<user_lang>`
- **Maximum**: `large-v3`, `beam_size=5`, 2-pass + gap-fix (gaps > 0.5s, relaxed thresholds), `language=<user_lang>`

**Multiple languages:**
- **Light**: `large-v3-turbo`, `beam_size=1`, single-pass VAD on, `language=None`
- **Balanced**: `large-v3-turbo`, `beam_size=5`, single-pass VAD on, `language=None`
- **Thorough**: `large-v3` (NOT turbo), `beam_size=5`, 2-pass, `language=None`
- **Maximum**: `large-v3`, `beam_size=5`, 2-pass + gap-fix, `language=None`

### Hallucination Filter (gap-fill / gap-fix passes)

Skip any segment where:
- `avg_logprob < -1.0`
- `no_speech_prob > 0.8`
- text is empty or fewer than 3 characters

### Output Files (all profiles)

- `.txt` — timestamped text: `[HH:MM:SS --> HH:MM:SS] text`
- `.srt` — standard subtitles
- `.json` — full metadata with word-level timestamps, per-segment `detected_language`

Run script in background via `run_in_background=true`.

### Script Architecture (CRITICAL — save before cleanup)

The generated script MUST follow this execution order:

1. Run transcription passes (1-3 depending on profile)
2. Set `speaker: "UNKNOWN"` on all segments
3. Merge & deduplicate (remove 3+ consecutive identical texts)
4. **SAVE all outputs** (.txt, .srt, .json) and **print QC summary**
5. Unload whisper model — wrap in try/except: `try: del model` then `torch.cuda.empty_cache()` (may crash on some systems)
6. Attempt diarization (Step 4) — if successful, re-save all three output files with real speaker labels

**Why this order is mandatory**: `del model` and `torch.cuda.empty_cache()` can crash the Python process on Windows (exit code 127). pyannote may fail (no HF_TOKEN, import error, VRAM OOM). If outputs are saved only after these steps, a crash loses all transcription work. This happened on 23 Mar 2026 — three complete transcription passes (232 segments, ~2 min GPU) were lost twice because saves came after cleanup. Save FIRST, enhance SECOND.

## Step 4 — Speaker Diarization (POST-SAVE enhancement)

**Pre-condition**: All outputs (.txt, .srt, .json) must already be saved with `speaker: "UNKNOWN"` from Step 3. Diarization is an enhancement — if it fails, the transcript is complete and usable.

After outputs are saved and whisper model is unloaded:

1. Wrap the entire diarization block in try/except. On any failure, print a message and continue — outputs are already saved.
2. Run **pyannote.audio** diarization pipeline on the audio file (CUDA).
3. Align speaker labels with transcript segments by timestamp overlap.
4. Add `speaker` field to each segment in the `.json` (e.g., `"SPEAKER_01"`, `"SPEAKER_02"`).
5. **Re-save** all three output files (.txt, .srt, .json) with updated speaker labels.
6. If pyannote fails or is unavailable, outputs remain saved with `speaker: "UNKNOWN"` — note this in QC output.

**VRAM management is critical**: whisper and pyannote cannot coexist in 8 GB VRAM simultaneously.

## Step 5 — Monitor Every 2 Minutes (MANDATORY)

After launching script in background:

1. Check `TaskOutput` every **2 minutes** — no exceptions, even for 1-hour transcriptions.
2. Look for: tracebacks, CUDA OOM, process hangs (no new output for >3 min), stalled progress.
3. Report brief status to user at each check (e.g., "Pass 1 at segment 150, ~12 min processed").
4. If error: diagnose, fix script, re-run.
5. Continue until script completes.

## Step 6 — Post-Transcription QC Loop (MANDATORY)

**Quality standard: transcripts must be reliable enough for deep legal/financial/strategic analysis.**

### QC Checks

| # | Check | PASS Criteria | Fix Strategy |
|---|-------|---------------|-------------|
| 1 | **Coverage** >= 95% | (speech segments + confirmed silence where `no_speech_prob > 0.9`) / `total_duration` | Run gap-fix on remaining gaps > 0.5s with relaxed thresholds |
| 2 | **Max gap** < 2s | Largest inter-segment gap (excluding confirmed silence) | Targeted re-transcription on specific gap with VAD off |
| 3 | **No hallucinations** | No 3+ consecutive identical texts | Remove duplicates, keep first occurrence |
| 4 | **Boundaries OK** | First seg starts <= 30s, last ends <= 30s from end | Re-transcribe first/last 60s with VAD off |
| 5 | **Segment count** proportional | ~5-25 segments/minute | Informational — report to user |
| 6 | **Low-confidence** < 15% | Segments with `avg_logprob < -1.0` | Informational — report to user |
| 7 | **Word confidence** | Flag segments where avg word probability < 0.5 | Report for manual review |
| 8 | **Semantic spot-check** | Agent reads 10 evenly-spaced excerpts and flags garbled/nonsensical text | Agent fixes or flags for user |

### Fix Loop

```
for iteration in range(1, 4):
    run QC checks
    if all pass → break
    apply fixes (remove hallucinations → gap-fix → boundary fix)
    re-merge, re-save all outputs
    if no fixes improved anything → report remaining issues, break
if still failing after 3 rounds → report to user, ask whether to proceed
```

**Gap-fix relaxed thresholds**: `logprob < -1.5`, `no_speech > 0.9`, gap threshold `0.5s`, try `beam_size=5` then `1`.

## Step 7 — Delete Original Recording

**Only after QC passes** (or user accepts remaining issues):

1. Confirm with user: "QC passed. Delete original recording at `<path>`?"
2. This is a **Red operation** per safety-guard — always confirm unless PermitAll active.
3. If declined, leave file and note in chronology.

## Step 8 — File & Update Context

### Output Location

Save to `operations/Records/YYYY-MM-DD_<Description>/` containing `.txt`, `.srt`, `.json`.

### FileAnalysisCurrent.md Entry

```
### DOC-### | Transcript: <Description>
- **Date**: YYYY-MM-DD
- **Source**: <original filename> (deleted after transcription)
- **Location**: <output directory path>
- **Files**: .txt, .srt, .json
- **Language**: <lang> | **Speakers**: <N> identified
- **Duration**: <X> min | **Coverage**: <X>%
- **Model**: faster-whisper <model> | **Profile**: <name>
- **QC**: PASS | **Diarization**: Yes/No
- [Project: <Name>]
```

### ChronologyCurrent.md Entry

```
### YYYY-MM-DD
- [DOC-###] Audio transcribed: <Description>. <duration> min, <model>, <profile>, coverage <X>%, <N> speakers. Original deleted. [Project: <Name>]
```
