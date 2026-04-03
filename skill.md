---
name: edit-greek-reel
description: Edit a raw talking-head video into a polished short-form reel with karaoke subtitles. Trims silence, adds Manrope Bold subtitles, zoom effects, SFX, and image overlays. Supports any language. Usage - /edit-greek-reel <path-to-video> [options]
argument-hint: <path-to-video.MOV> [--lang en] [--crop-top 20] [--no-images] [--manual-text "your script here"]
---

# Greek Reel Video Editor — Artemis Codes

You are a senior short-form video editor. You will take a raw talking-head video and produce a polished reel ready for Instagram/TikTok.

**Input**: $ARGUMENTS

## Pipeline Overview

The editing pipeline has 3 passes:
1. **Trim + Crop + Scale** — Cut silence, remove retakes, crop to 9:16 (object-cover, never stretch)
2. **Subtitles + Zoom + Image Overlays** — Burn karaoke-style subs, add subtle zooms and logo/image overlays
3. **Mix SFX** — Layer sound effects on key moments

## Step 1: Analyze the Video

1. Run `ffprobe` to get resolution, duration, rotation, codec info
2. Check orientation — if rotation is 90/270, the video is portrait (swap w/h)
3. Detect silence gaps with: `ffmpeg -i <input> -vn -af "silencedetect=noise=-30dB:d=0.5" -f null -`

## Step 2: Transcribe

### 2a. Determine Language

If the user provided `--lang <code>` (e.g., `--lang en`, `--lang el`, `--lang es`), use that language code directly.

Otherwise, **ask the user** which language the video is in before transcribing. Common options:
- `en` — English
- `el` — Greek
- `es` — Spanish
- `fr` — French
- `de` — German

The user can also type any [Whisper language code](https://github.com/openai/whisper#available-models-and-languages).

### 2b. Download Whisper Model (if needed)

The Whisper medium model (~1.5 GB) must be downloaded before first use. **Download it as a separate step** before transcribing, so the user can see progress and it doesn't timeout during transcription:

```python
import whisper
print("Downloading Whisper medium model (this may take a few minutes on first run)...")
model = whisper.load_model("medium")
print("Model ready.")
```

Run this in a script with a generous timeout (5+ minutes). If the download fails or times out, retry — the download resumes from where it left off.

### 2c. Transcribe

```python
result = model.transcribe(audio_path, language=LANG_CODE, word_timestamps=True, condition_on_previous_text=True)
```

Save transcript to `transcript.json` in the same directory. Print the full transcript and word timestamps for review.

## Step 3: Proofread the Transcription

**CRITICAL**: Whisper makes mistakes, especially with:
- Brand/tool names in any language (e.g., "Cloud Code" → "Claude Code", "CacheSource" → "Cursor", "Artemis Coe" → "Artemis Codes")
- Homophones and near-misses (e.g., "lucky" → "lowkey", "dragon" → "dragging")
- Spelling errors in the target language
- Merged or split words

Review the transcript yourself and fix obvious errors. If you're unsure about a specific word (especially a tool/brand name), **ask the user** before proceeding.

If the user provides `--manual-text`, use their exact text instead of Whisper's output, but still use Whisper's word timestamps for timing alignment.

## Step 4: Build Segments & Timed Words

Based on the silence detection and word timestamps:

1. Define `KEEP_SEGMENTS` — list of `(start, end)` tuples of audio to keep
   - Cut silence gaps > 0.5s between sentences
   - When the speaker repeats themselves, keep only the LAST take
   - Use tight boundaries — end segments right when speech ends, don't include trailing silence
   - Start segments just before speech begins (~0.05s padding)

2. Define `TIMED_WORDS` — list of `(word, start, end)` with the CORRECTED text mapped to Whisper timestamps

3. Recalculate all timestamps relative to the trimmed output

## Step 5: Configure Effects

### Subtitles (Karaoke Style)
- Font: Manrope Bold (search for `Manrope-Bold.otf` or `Manrope-Bold.ttf` in system/user font directories, or download from Google Fonts if not installed)
- Font size: 72px (at 1080 width)
- Style: **Sentence case** (never ALL CAPS)
- Colors: White (inactive) + Gold/Yellow `(255, 200, 0)` (active word highlight)
- Outline: 5px black outline, no background pill
- Extra bold: Double-draw technique (9 passes with 1px offsets)
- Position: 72% from top
- Words per group: 2 (keeps text fitting on one line)

### Zoom Effects (Subtle)
- Maximum 5 zoom triggers per video
- Zoom factor: 1.08–1.10x (never more than 1.12x — avoid making viewer dizzy)
- Duration: 0.35–0.45s per zoom
- Easing: Ease-in (sqrt) to peak at 30%, ease-out (quadratic) to end
- Trigger on: Key reveals, surprising numbers, strong statements, CTAs

### Sound Effects
- **NEVER repeat the same SFX file twice in one video**
- This skill ships with pre-trimmed SFX in its `audios/` directory (relative to this skill.md file):
  - `trimmed_whoosh.mp3` — transitions, reveals
  - `trimmed_cash.mp3` — money/price mentions
  - `trimmed_fah.mp3` — emphasis, strong statements
  - `trimmed_click.mp3` — tool mentions
  - `trimmed_bubble_pop.mp3` — light reveals
  - `trimmed_riser.mp3` — builds, anticipation
- **Locating the SFX files** — check these locations in order:
  1. The skill's base directory `audios/` folder (provided at invocation as `Base directory for this skill: <path>`)
  2. The video's parent directory for an `audios/` folder (user may have added custom SFX there)
  3. **If audios/ is missing** (e.g., package managers like ClawHub may not include binary assets): download them from the repository:
     ```bash
     REPO_URL="https://raw.githubusercontent.com/artemisln/edit-greek-reel/main/audios"
     SKILL_DIR="<skill base directory>"
     mkdir -p "$SKILL_DIR/audios"
     for f in trimmed_whoosh.mp3 trimmed_cash.mp3 trimmed_fah.mp3 trimmed_click.mp3 trimmed_bubble_pop.mp3 trimmed_riser.mp3; do
       curl -sL "$REPO_URL/$f" -o "$SKILL_DIR/audios/$f"
     done
     ```
     On Windows, use Python `urllib` instead of `curl`.
- Also check the video's parent directory for an `audios/` folder — the user may have added custom SFX there
- If new untrimmed audio files exist, trim leading silence first:
  ```
  ffmpeg -i input.mp3 -ss <silence_end> -acodec libmp3lame -q:a 2 trimmed_output.mp3
  ```
- Volume: 0.15–0.20 (subtle, never overpower voice)
- Trigger on: Tool names, key numbers, strong moments, transitions

### Image Overlays
- Check `images/` directory for available logos, screenshots, memes
- Display above the speaker's head area (centered, ~15% from top)
- Logo size: 200px max
- Meme/screenshot size: 500px max
- Animation: Pop-in (ease-out over first 15%) and pop-out (over last 15%)
- Duration: 1.8–2.5s per image
- Trigger on: When the speaker mentions the tool/concept the image represents
- Each image triggers only once
- Convert SVGs to PNG first if needed (use `cairosvg`)

## Step 6: Video Processing

### Crop (Object-Cover, Never Stretch)
- Target: 1080x1920 (9:16)
- If `--crop-top N` is specified, remove N% from the top before fitting
- Always crop to fit the target ratio (like CSS `object-fit: cover`), never scale-to-fit (which would stretch/distort)
- Center the crop horizontally; for vertical, bias toward bottom-center (keep the speaker's face)

### Processing Pipeline (Python + ffmpeg + Pillow)

**Pass 1: Trim + Crop + Scale (ffmpeg)**
- Build a complex filter: trim each segment, concat, crop to 9:16, scale to 1080x1920
- Concat uses interleaved stream ordering: `[v0][a0][v1][a1]...concat=n=N:v=1:a=1`
- Output: temp_trimmed.mp4 (libx264, crf 18, aac 192k, 30fps)

**Pass 2: Subtitles + Zoom + Images (Pillow frame-by-frame)**
- Decode trimmed video to raw RGBA frames via ffmpeg pipe
- For each frame:
  1. Apply zoom effect if active (center-crop + resize)
  2. Composite image overlay if active (with pop animation)
  3. Composite subtitle overlay
- Encode back to mp4 via ffmpeg pipe

**Pass 3: Mix SFX (ffmpeg)**
- Overlay all SFX using `adelay` + `amix` filter
- Use `normalize=0` to prevent volume pumping
- Copy video stream, re-encode audio only

### Output
- Save as `final_<name>.mp4` in the same directory as the input
- Print summary: original duration → final duration, number of effects applied
- Clean up temp files

## Important Rules

1. **Never stretch video** — always crop to fit (object-cover behavior)
2. **Proofread before burning subtitles** — Whisper WILL get tool names wrong
3. **Ask the user** if unsure about a word, especially brand/tool names
4. **Sentence case only** — never ALL CAPS subtitles
5. **No background pill** behind subtitles — outline only
6. **Unique SFX** — never use the same sound file twice in one video
7. **Subtle zooms** — 1.08-1.10x max, 5 per video max
8. **Tight cuts** — trim silence aggressively, the reel should feel fast-paced
9. **Cache transcript** — if `transcript.json` exists, reuse it (skip re-transcription)
10. **Keep the last take** — when the speaker repeats, always keep the final version
