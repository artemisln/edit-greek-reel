# Greek Reel Video Editor — Artemis Codes

A Claude Code custom skill that turns raw talking-head videos into polished Instagram Reels / TikToks with Greek karaoke-style subtitles. No manual editing needed — just point it at your video and go.

## What it does

- **Transcribes** your video using OpenAI Whisper (with automatic proofreading)
- **Trims silence** and dead air between sentences
- **Removes retakes** — keeps only your last take when you repeat yourself
- **Burns karaoke-style subtitles** — word-by-word highlight in Manrope Bold
- **Adds subtle zoom effects** on key moments (1.08-1.10x, not dizzy-inducing)
- **Overlays images/logos** above your head when you mention tools or brands
- **Mixes sound effects** — whoosh, cash register, pop, etc. on key words
- **Crops to 9:16** (object-cover, never stretches your video)

## Before / After

| | Before | After |
|---|---|---|
| Duration | 90s raw footage | ~38s tight edit |
| Subtitles | None | Karaoke-style, sentence case |
| Silence | 50+ seconds of gaps | Cut to zero |
| Effects | None | Zooms, SFX, image overlays |

## Installation

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed
- Python 3.10+
- ffmpeg (`brew install ffmpeg` on macOS)
- [Manrope font](https://fonts.google.com/specimen/Manrope) installed (Bold weight required)

### Install the skill

```bash
# Clone or download this repo
git clone https://github.com/YOUR_USERNAME/edit-greek-reel-skill.git

# Copy to your Claude Code skills directory
cp -r edit-greek-reel ~/.claude/skills/edit-greek-reel
```

### Install Python dependencies

```bash
pip install openai-whisper Pillow cairosvg
```

## Usage

In Claude Code, run:

```
/edit-greek-reel ~/path/to/your/raw_video.MOV
```

### Options

```
/edit-greek-reel ~/video.MOV --crop-top 20          # Crop 20% from the top
/edit-greek-reel ~/video.MOV --no-images            # Skip image overlays
/edit-greek-reel ~/video.MOV --manual-text "..."    # Use your own script text instead of Whisper
```

### Adding custom SFX

Drop `.mp3` files into an `audios/` folder next to your video. The skill will auto-detect and trim leading silence from them.

The skill ships with 6 bundled SFX:
- `trimmed_whoosh.mp3` — transitions, reveals
- `trimmed_cash.mp3` — money/price mentions
- `trimmed_fah.mp3` — emphasis, strong statements
- `trimmed_click.mp3` — tool mentions
- `trimmed_bubble_pop.mp3` — light reveals
- `trimmed_riser.mp3` — builds, anticipation

### Adding image overlays

Create an `images/` folder next to your video and add logos/screenshots/memes as `.png`, `.webp`, or `.svg` files. Name them after what they represent (e.g., `cursor.png`, `github_copilot.png`). The skill will overlay them when you mention the corresponding tool.

## How it works

### 3-Pass Pipeline

1. **Pass 1: Trim + Crop + Scale** (ffmpeg)
   - Cuts silence gaps and retakes
   - Crops to 9:16 ratio (object-cover style)
   - Scales to 1080x1920

2. **Pass 2: Subtitles + Zoom + Images** (Python/Pillow, frame-by-frame)
   - Renders karaoke subtitles with word-by-word highlighting
   - Applies smooth zoom effects with ease-in/ease-out
   - Composites image overlays with pop-in/pop-out animation

3. **Pass 3: Mix SFX** (ffmpeg)
   - Layers sound effects at precise timestamps
   - Never repeats the same SFX in one video

### Subtitle Style

- Font: Manrope Bold, 72px
- Inactive words: White with black outline
- Active word: Gold/Yellow highlight
- No background pill — outline only
- 2 words per line (always fits on screen)
- Sentence case (never ALL CAPS)

## File Structure

```
edit-greek-reel/
  skill.md          # The Claude Code skill definition
  audios/           # Bundled sound effects
    trimmed_whoosh.mp3
    trimmed_cash.mp3
    trimmed_fah.mp3
    trimmed_click.mp3
    trimmed_bubble_pop.mp3
    trimmed_riser.mp3
  README.md         # This file
```

## Tips

- Record in **portrait mode** (9:16) for best results
- Speak naturally — the skill handles retakes and pauses
- If Whisper gets a word wrong, the skill will ask you before proceeding
- For best subtitle timing, speak clearly with brief pauses between sentences
- Add tool logos to `images/` if you're reviewing or mentioning specific tools

## Credits

Built by [Artemis Codes](https://instagram.com/artemiscodes) using Claude Code.

## License

MIT — use it however you want.
