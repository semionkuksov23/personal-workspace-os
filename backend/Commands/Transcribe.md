# Command: Transcribe

## Purpose
Transcribe audio/video recordings using faster-whisper on CUDA with quality profiles, optional speaker diarization, mandatory monitoring, and automated QC — producing transcripts reliable enough for deep legal, financial, and strategic analysis.

## Inputs
- Audio/video file path (`.mp4`, `.m4a`, `.mp3`, `.wav`, `.ogg`, `.webm`, `.mkv`, `.flac`)
- Language(s) — asked in Step 1
- Speaker diarization on/off (+ optional speaker count) — asked in Step 1
- Description/label — asked in Step 1
- Quality profile — chosen in Step 2

---

## ⚠️ Architecture rule (read first) — transcription and diarization are TWO SEPARATE PROCESSES

Diarization (speaker labelling) MUST run in its **own Python process, after the
transcription process has fully exited** — never inline in the transcription script.

**Why (structural, not stylistic):** whisper's GPU teardown — `del model` /
`torch.cuda.empty_cache()` — can **hard-crash the Python process on Windows (exit
code 127)**. If diarization sits inline (same process, after that teardown), the
crash silently skips speaker labelling: the transcript saves fine but every line
stays `speaker: "UNKNOWN"`. This is not hypothetical — it wiped the speaker labels
on a 43-minute meeting on **3 Jun 2026** even though the transcript itself was
perfect. Running diarization as a second process means the whisper-teardown crash
can never reach it: the fresh process gets a clean CUDA/CPU context with the GPU
already freed by the exited transcription process.

So the pipeline is always two phases:
- **Phase A — transcription** (`backend/Scripts/transcribe_<STEM>.py`): whisper + save
  + keep the 16 kHz WAV + exit. Never imports pyannote. Never calls
  `del model`/`empty_cache()`.
- **Phase B — diarization** (`backend/Scripts/diarise_existing.py`, only if `DIARIZE`):
  a separate process, run after Phase A exits, that adds speaker labels to the
  already-saved transcript.

---

## Step 1 — Validate, Ask Language & Speakers

1. Validate file exists and has a supported extension.
2. Get duration via `ffprobe -v error -show_entries format=duration -of csv=p=0 "<file>"`.
3. **Ask user**: "What language(s) are present in the recording?"
   - Single language → set `language` param (e.g., `"en"`, `"ru"`)
   - Mixed languages → set `language=None` (auto-detect per segment)
4. **Ask user**: "Enable speaker diarization? (labels each line with who spoke — SPEAKER_01, SPEAKER_02, …)"
   - This is **optional and OFF by default**. State the trade-offs:
     - Adds little time now — pyannote runs on **GPU** (CUDA torch cu128 installed 2026-06-13), so diarization adds only ~0.03–0.05x of the audio duration (a ~2-hour call diarizes in ~4 minutes). Transcription itself also uses CUDA (whisper via ctranslate2). Without a CUDA torch build it falls back to CPU (~0.5–1.0x).
     - Requires `pyannote.audio` (installed) **and** a Hugging Face `HF_TOKEN` with the gated pyannote models accepted — see **Diarization Setup** below.
   - If **yes**, also ask: "How many distinct speakers? (leave blank to auto-detect)" → set `num_speakers` when known (improves accuracy).
   - If **yes but `HF_TOKEN` is not set**, point the user to the Diarization Setup section and ask whether to (a) proceed anyway, producing `speaker: "UNKNOWN"`, or (b) pause until the token is configured.
   - Record the result as a `DIARIZE` flag (True/False). It controls whether Phase B (Step 4) runs at all.
5. **Ask user**: Short description/label for file naming (or derive from filename if obvious).

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

**Note on diarization time:** the estimates above are for transcription (Phase A) only. If diarization was enabled in Step 1, the separate Phase-B process now runs on **GPU** (CUDA torch cu128, installed 2026-06-13) and adds only **~0.03–0.05x duration** (minutes) to the total wall-clock — a ~2-hour call diarizes in ~4 min. Because Phase B now also uses the GPU, run it AFTER Phase A has exited (sequential GPU use); do not run two GPU diarizations on the same card at once.

## Step 3 — Generate & Run Phase-A Transcription Script

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
        "detected_language": lang,
        "speaker": "UNKNOWN"
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

- `.txt` — timestamped text: `[HH:MM:SS --> HH:MM:SS] [SPEAKER] text`
- `.srt` — standard subtitles
- `.json` — full metadata with word-level timestamps, per-segment `detected_language`, per-segment `speaker`

Run script in background via `run_in_background=true`.

### Phase-A execution order (CRITICAL — save, keep the WAV, then exit clean)

The generated transcription script MUST follow this order, and MUST NOT do diarization:

1. Extract a 16 kHz mono WAV once (`_audio_16k.wav` in the output dir) and reuse it for both whisper passes.
2. Run transcription passes (1-3 depending on profile).
3. Set `speaker: "UNKNOWN"` on all segments.
4. Merge & deduplicate (remove 3+ consecutive identical texts).
5. **SAVE all outputs** (.txt, .srt, .json) and **print the QC summary**.
6. **Keep `_audio_16k.wav` on disk** (Phase B reuses it) and **print a `[transcribed]` marker as the last line, then let the process exit.**
   - Do **NOT** call `del model` / `torch.cuda.empty_cache()` — that is the exit-127
     crash trigger and it is unnecessary: when the process exits, the OS reclaims all
     GPU memory. A crash *during interpreter shutdown* is harmless because every output
     is already on disk.
   - The transcription script **never imports pyannote** and **never touches diarization**.

**Success signal for an orchestrator/monitor**: the presence of the output files +
the `[transcribed]` line — **not** exit code 0 (the process may still segfault during
CUDA shutdown after everything is saved; that is expected and harmless).

**Why save-first + keep-WAV is mandatory**: a crash during model teardown must never
lose transcription work (this happened on 23 Mar 2026 — three complete passes lost
twice because saves came after cleanup), and Phase B needs the decoded WAV to avoid
re-decoding. Save FIRST, keep the WAV, exit.

## Step 4 — Speaker Diarization (Phase B — SEPARATE PROCESS, OPTIONAL)

**Runs only when `DIARIZE` is True** (user opted in at Step 1), and **only after the
Phase-A transcription process has fully exited.** If `DIARIZE` is False, skip this
step entirely — the transcript is already complete with `speaker: "UNKNOWN"`, and
pyannote is never imported anywhere.

Diarization runs as its **own Python process** — never inline in the transcription
script (see the Architecture rule at the top for why). Because it is a fresh process
with the GPU already freed by the exited Phase-A process, the whisper-teardown crash
cannot reach it.

**Pre-conditions**:
- Phase A saved all outputs (.txt, .srt, .json) with `speaker: "UNKNOWN"`.
- Phase A left `_audio_16k.wav` on disk in the output dir.
- The Phase-A process has exited (GPU freed).

**How to run** — invoke the canonical standalone diarizer as a separate background process:

```
python backend/Scripts/diarise_existing.py <output_dir> <stem> [num_speakers]
```

It loads `<output_dir>/<stem>.json` + `<output_dir>/_audio_16k.wav`, runs pyannote
(CUDA if a CUDA torch build is present, else CPU — falls back to CPU automatically),
aligns speaker turns to segments by max-overlap, re-saves all three outputs with real
speaker labels, deletes the WAV, and prints `[done]`. If it fails, the transcript is
already complete and usable with `speaker: "UNKNOWN"`.

**Success signal**: `[done]` in its log (and `[diarization] complete: N speakers`) —
not the exit code.

**Serialization**: pyannote now runs on **GPU** (CUDA torch cu128). Run Phase B
AFTER Phase A (whisper) has exited so the two share the GPU sequentially (peak
diarization VRAM ~2.9 GB); do not run two GPU diarizations on the same card at once.

### Canonical Phase-B diarizer (pyannote 4.x — torchcodec-free) — `backend/Scripts/diarise_existing.py`

This standalone script IS Phase B. Generated transcription scripts call it as a
separate process; never re-embed diarization into the transcription script. The
installed pyannote is **4.x** (`token=` not `use_auth_token=`); its default decoder
(`torchcodec`) cannot load on this machine, so we feed an in-memory waveform decoded
from the saved WAV.

```python
"""
Standalone speaker-diarisation pass for an ALREADY-transcribed meeting.
Decoupled from whisper so a whisper GPU-cleanup crash cannot block labelling.
Usage: python diarise_existing.py <output_dir> <stem> [num_speakers]
"""
import os, sys, json
sys.stdout.reconfigure(encoding="utf-8")

# cuBLAS DLL fix (MUST precede torch/pyannote import)
_cublas_dir = os.path.join(os.environ["LOCALAPPDATA"],
    r"Python\pythoncore-3.14-64\Lib\site-packages\nvidia\cublas\bin")
if os.path.isdir(_cublas_dir):
    os.environ["PATH"] = _cublas_dir + os.pathsep + os.environ["PATH"]
    os.add_dll_directory(_cublas_dir)

OUTPUT_DIR   = sys.argv[1]
STEM         = sys.argv[2]
NUM_SPEAKERS = int(sys.argv[3]) if len(sys.argv) > 3 else None
AUDIO_WAV    = os.path.join(OUTPUT_DIR, "_audio_16k.wav")
base         = os.path.join(OUTPUT_DIR, STEM)

def format_ts(s):
    h=int(s//3600); m=int((s%3600)//60); sec=int(s%60); ms=int((s-int(s))*1000)
    return f"{h:02d}:{m:02d}:{sec:02d},{ms:03d}"
def format_ts_txt(s):
    h=int(s//3600); m=int((s%3600)//60); sec=int(s%60)
    return f"{h:02d}:{m:02d}:{sec:02d}"

with open(base + ".json", encoding="utf-8") as f:
    data = json.load(f)
all_segments = data["segments"]

def save_outputs():
    with open(base+".txt","w",encoding="utf-8") as f:
        for s in all_segments:
            f.write(f"[{format_ts_txt(s['start'])} --> {format_ts_txt(s['end'])}] [{s['speaker']}] {s['text']}\n")
    with open(base+".srt","w",encoding="utf-8") as f:
        for i,s in enumerate(all_segments,1):
            f.write(f"{i}\n{format_ts(s['start'])} --> {format_ts(s['end'])}\n[{s['speaker']}] {s['text']}\n\n")
    data["segments"]=all_segments; data["diarized"]=True
    with open(base+".json","w",encoding="utf-8") as f:
        json.dump(data, f, ensure_ascii=False, indent=2)

import torch, soundfile as sf
from pyannote.audio import Pipeline

hf_token = os.environ.get("HF_TOKEN")
if not hf_token:
    print("HF_TOKEN not set — cannot diarise. Outputs keep speaker=UNKNOWN."); sys.exit(2)

audio_np, sr = sf.read(AUDIO_WAV, dtype="float32")
if audio_np.ndim > 1: audio_np = audio_np.mean(axis=1)
waveform = torch.from_numpy(audio_np).unsqueeze(0)

print("[diarization] loading pipeline ...", flush=True)
pipeline = Pipeline.from_pretrained("pyannote/speaker-diarization-community-1", token=hf_token)
device = "cuda" if torch.cuda.is_available() else "cpu"
try:
    pipeline.to(torch.device(device))
except Exception as e:
    print(f"[diarization] GPU move failed ({e}); CPU fallback", flush=True); device="cpu"; pipeline.to(torch.device("cpu"))
print(f"[diarization] running on {device} ...", flush=True)

diar_out = pipeline({"waveform": waveform, "sample_rate": sr}, num_speakers=NUM_SPEAKERS) if NUM_SPEAKERS else pipeline({"waveform": waveform, "sample_rate": sr})
diarization = diar_out.speaker_diarization
turns = [(t.start, t.end, spk) for t,_,spk in diarization.itertracks(yield_label=True)]
for seg in all_segments:
    best, best_ov = "UNKNOWN", 0.0
    for s,e,spk in turns:
        ov = max(0.0, min(seg["end"], e) - max(seg["start"], s))
        if ov > best_ov: best_ov, best = ov, spk
    seg["speaker"] = best

save_outputs()
n_spk = len({s["speaker"] for s in all_segments if s["speaker"] != "UNKNOWN"})
print(f"[diarization] complete: {n_spk} speakers. Outputs re-saved.", flush=True)
try: os.remove(AUDIO_WAV)
except OSError: pass
print("[done]", flush=True)
```

> Model note: pyannote v4 loads `"pyannote/speaker-diarization-community-1"`. The older `"pyannote/speaker-diarization-3.1"` is a v3 fallback; both require accepting their HF terms.

### Diarization Setup (one-time — required only if you enable diarization)

> Already completed on this machine (2026-06-01): HF_TOKEN saved + all three pyannote models accepted. Steps below are for reference / re-setup on a new machine.

Diarization needs three things. The Python packages are already installed; the other two are one-time manual steps only the user can perform:

1. **Python packages** — `pyannote.audio` (4.0.4) + `soundfile` installed via `pip` (already done on this machine). `torchcodec` is intentionally **bypassed**, so its load failure is harmless.
2. **Hugging Face token** — create a free account at https://huggingface.co, then generate a *read* token at https://huggingface.co/settings/tokens. Make it persistent so future transcription scripts can see it:
   ```
   setx HF_TOKEN "hf_xxxxxxxxxxxxxxxxx"
   ```
   `setx` only affects **future** processes — open a new terminal (and restart Claude Code) after running it.
3. **Accept the gated-model terms** (one click each, while logged into Hugging Face):
   - https://huggingface.co/pyannote/speaker-diarization-community-1  (the model pyannote v4 actually loads)
   - https://huggingface.co/pyannote/segmentation-3.0
   - https://huggingface.co/pyannote/speaker-diarization-3.1  (only needed for the v3 fallback)

   The pipeline returns 401/403 until the required models are accepted.

Speed: diarization runs on **GPU**. A CUDA torch build (cu128 for the RTX 5070, sm_120; Python 3.14: torch 2.11.0+cu128 + torchaudio + torchvision) was installed 2026-06-13, so Phase B is dramatically faster — benchmark RTF ~0.036 (a ~2-hour call diarizes in ~4 min vs ~1–2 h on CPU). If the CUDA torch build is ever removed, pyannote falls back to CPU automatically.

## Step 5 — Monitor Every 2 Minutes (MANDATORY — both phases)

After launching each script in background:

1. Check `TaskOutput` every **2 minutes** — no exceptions, even for 1-hour transcriptions. Monitor **both** Phase A (whisper) and Phase B (diarization).
2. Look for: tracebacks, CUDA OOM, process hangs (no new output for >3 min), stalled progress.
3. Report brief status to user at each check (e.g., "Pass 1 at segment 150, ~12 min processed"; "diarization running on cuda").
4. If error: diagnose, fix script, re-run.
5. **Phase A is complete when the `[transcribed]` marker is printed AND the output files exist** — do not rely on exit code 0 (CUDA-shutdown segfaults after save are expected). Then launch Phase B (if `DIARIZE`) as a separate process.
6. **Phase B is complete when `[done]` is printed** (and `[diarization] complete: N speakers`). On Phase-B failure, the transcript stays valid with `speaker: "UNKNOWN"`.

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
| 9 | **Speaker labels** (if `DIARIZE`) | `.txt`/`.json` carry real `SPEAKER_xx` labels, not all `UNKNOWN` | If all `UNKNOWN`, Phase B did not run/failed — re-run `diarise_existing.py` |

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

1. **Default — explicitly ask permission.** Confirm with user: "QC passed. Delete original recording at `<path>`?" The original recording is the irreplaceable source; never delete it without a clear go-ahead.
2. **Exception — pre-authorized deletion.** If the user already told you to delete the recording when they first invoked the command (e.g. "transcribe this and delete it after", "delete it once you're satisfied with the quality"), that standing instruction holds: once QC passes, **proceed to delete WITHOUT asking again** — do not re-prompt for something already authorized.
3. This is a **Red operation** per safety-guard — always confirm UNLESS the user pre-authorized deletion (step 2) or PermitAll is active.
4. If declined, or no authorization was given and the user does not respond, leave the file and note in chronology.

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
- **Language**: <lang> | **Speakers**: <N> identified (or "diarization off")
- **Duration**: <X> min | **Coverage**: <X>%
- **Model**: faster-whisper <model> | **Profile**: <name>
- **QC**: PASS | **Diarization**: Yes/No
- [Project: <Name>]
```

### ChronologyCurrent.md Entry

```
### YYYY-MM-DD
- [DOC-###] Audio transcribed: <Description>. <duration> min, <model>, <profile>, coverage <X>%, <N> speakers (or diarization off). Original deleted. [Project: <Name>]
```
