# Skill: YouTube Video Analysis

## Purpose

Provide a structured protocol for visually analysing YouTube videos by downloading, extracting frames, reading them with Claude's vision, and reporting findings. All temporary files are deleted after analysis to prevent disk bloat.

## When This Skill Applies

- The user asks to analyse, review, describe, watch, or inspect a YouTube video
- The user provides a YouTube URL and asks for visual analysis
- The user asks to search YouTube for a video and then analyse it
- The user asks what happens visually in a video

## Relationship to workspace-skill.md

This skill **extends** the workspace with a new capability (video visual analysis). It does not override any existing filing rules. Temporary files live in `operations/Operations/VideoFrames/` and are always cleaned up. If the user asks to save results, they go to `operations/Research/` following the web-research pattern.

## Prerequisites

- `yt-dlp` — installed as Python module (`py -m yt_dlp`)
- `ffmpeg` — available on PATH

## Protocol

### Step 1: Validate Input

If the user provides a YouTube URL (containing `youtube.com` or `youtu.be`), use it directly.

If the user describes a video but does not provide a URL, search YouTube:
```bash
py -m yt_dlp --print title --print id --print duration --no-download "ytsearch5:<query>"
```
Present the results (title, duration) and ask the user to pick one, unless PermitAll is active (in which case pick the best match).

If the user provides a local video file path, skip to Step 4 (extract frames). Never delete a user-provided local file.

### Step 2: Get Metadata

Retrieve title, ID, and duration:
```bash
py -m yt_dlp --print title --print id --print duration --no-download "<URL>"
```

Record the video ID for use as the working folder name.

### Step 3: Calculate Frame Rate

Choose an adaptive extraction rate based on video duration:

| Duration | fps value | Rationale |
|----------|-----------|-----------|
| < 30 seconds | `fps=1` | Short clip, capture every second |
| 30s – 2 minutes | `fps=1/2` | One frame every 2 seconds |
| 2 – 10 minutes | `fps=1/5` | One frame every 5 seconds |
| 10 – 30 minutes | `fps=1/10` | One frame every 10 seconds |
| > 30 minutes | `fps=1/15` | One frame every 15 seconds |

**Hard cap: 30 frames maximum.** If the calculated number of frames exceeds 30, reduce the fps value accordingly.

**Videos over 1 hour:** Warn the user that full analysis will produce many frames and suggest providing a time range. If the user provides `-ss` (start) and/or `-to` (end) values, pass them to both yt-dlp and ffmpeg.

### Step 4: Download Video

```bash
py -m yt_dlp -f "best[ext=mp4][height<=720]" -o "<workspace>/operations/Operations/VideoFrames/<video_id>/_temp_video.mp4" --no-playlist --quiet "<URL>"
```

- If `operations/Operations/VideoFrames/` does not exist, create it.
- If a stale `VideoFrames/<video_id>/` directory exists from a previous failed run, delete it first.
- If download fails, report the error to the user and clean up any partial files.

### Step 5: Extract Frames

```bash
ffmpeg -i "<workspace>/operations/Operations/VideoFrames/<video_id>/_temp_video.mp4" -vf "fps=<rate>,scale=1024:-1" -q:v 2 -y "<workspace>/operations/Operations/VideoFrames/<video_id>/frame_%04d.jpg"
```

After extraction, count the frames produced. If more than 30:
1. Select 30 evenly-spaced frames from the set.
2. Delete the rest.
3. Rename remaining frames sequentially as `frame_0001.jpg` through `frame_0030.jpg`.

### Step 6: Delete Video File

Immediately delete the downloaded video before starting analysis:
```bash
rm "<workspace>/operations/Operations/VideoFrames/<video_id>/_temp_video.mp4"
```

This keeps disk usage to only the extracted JPEG frames during analysis.

For local video files provided by the user: **do NOT delete the original**. Only delete the frames after analysis.

### Step 7: Analyse Frames

Read the extracted frames using the Read tool in batches of up to 15 images at a time. For each frame, note:
- Frame number and approximate timestamp (calculated from frame number and fps rate)
- Key visual content: people, objects, text on screen, diagrams, settings, actions
- Changes from the previous frame(s)

Focus the analysis on the user's specific question. If no specific question was asked, provide a general visual summary.

### Step 8: Report

Present a structured report:

```
## Video Analysis: <Title>
**URL:** <url>
**Duration:** <duration>
**Frames analysed:** <count>

### Timeline

| Timestamp | Frame | Description |
|-----------|-------|-------------|
| 0:00 | 1 | ... |
| 0:05 | 2 | ... |
| ... | ... | ... |

### Summary
<Paragraph summarising the visual content, focused on the user's question>

### Key Observations
- <Notable finding 1>
- <Notable finding 2>
```

### Step 9: Mandatory Cleanup

After reporting, **always** delete all temporary files:
```bash
rm -rf "<workspace>/operations/Operations/VideoFrames/<video_id>/"
```

Then attempt to remove the parent directory if empty:
```bash
rmdir "<workspace>/operations/Operations/VideoFrames/" 2>/dev/null
```

**This step is not optional.** Even if the analysis fails or the user interrupts, clean up before finishing.

### Step 10: Save Results (Only If Requested)

If the user asks to save the analysis:

Save to: `operations/Research/<YYYY-MM-DD>_Video_<Topic_Slug>.md`

Include metadata header:
```
---
CapturedDate: YYYY-MM-DD
Topic: Video analysis — <title>
Sources:
  - URL: <youtube_url>
    AccessedDate: YYYY-MM-DD
    VideoID: <id>
    Duration: <duration>
Project: <project tag(s)>
Relevance: <one-sentence summary>
---
```

Then follow the standard research filing protocol:
1. Create `FileAnalysisCurrent.md` entry with `DOC-###` ID
2. Add `ChronologyCurrent.md` entry: `YYYY-MM-DD — Video analysis captured: <topic> — <file path>`

## Edge Cases

- **Download failure:** Report error, clean up partial files in VideoFrames/, do not retry automatically.
- **No frames extracted:** Report that ffmpeg produced no output, check if the video format is supported, clean up.
- **Stale VideoFrames directory found at start:** Delete it before proceeding with new analysis.
- **User provides time range:** Pass `-ss <start>` and `-to <end>` to both yt-dlp (via `--download-sections`) and ffmpeg.
- **Age-restricted or private video:** Report the restriction to the user. Do not attempt to bypass.

## Safety

- All operations are **Green** — creating and deleting own temporary files in `operations/Operations/VideoFrames/`.
- No modifications to Inbox, Outbox, Records, or Admin.
- The video file is deleted before analysis begins; frames are deleted after analysis completes.
- Disk usage is transient: only JPEG frames exist during the analysis window.

## Output

- Visual analysis report (always, in chat)
- Research file: `operations/Research/<YYYY-MM-DD>_Video_<Topic_Slug>.md` (only if user asks to save)
- FileAnalysis + Chronology entries (only if saving)
