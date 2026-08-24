---
name: youtube-to-text
description: Download a YouTube video (or local audio/video file), transcribe its speech to clean text, then summarize it. Strong Hebrew (via ivrit.ai Whisper finetunes) and English. Two engines — a free/offline local model (faster-whisper) and an optional ElevenLabs Scribe v2 cloud engine for top accuracy. Use when the user wants to "read"/summarize a video, get a transcript, "תמלל את הסרטון", "מה נאמר בסרטון", "transcribe this", "summarize this YouTube video", or pastes a YouTube link and wants its content. Local engine runs on Apple Silicon — free, private, offline after the first model download.
---

# YouTube to text

Turns a YouTube link (or a local media file) into a clean text transcript so Claude can read,
quote, and summarize what was said. Two transcription engines:

- **`local`** (default) — `faster-whisper` on-device, **free, private, offline**. Hebrew uses
  the **ivrit.ai** Whisper finetunes (state-of-the-art open Hebrew ASR), English uses Whisper
  large-v3.
- **`elevenlabs`** — ElevenLabs **Scribe v2** cloud API. **Highest Hebrew accuracy** (also gets
  English brand/tech terms right and can fix genuine Hebrew word errors the local model misses),
  plus built-in speaker diarization. Costs money and uploads the audio to ElevenLabs.

Rule of thumb: default to `local`; reach for `--engine elevenlabs` when Hebrew accuracy really
matters (publishing the transcript, many garbles, or you need diarization).

## When to use
- The user pastes a YouTube URL and wants its content read / summarized / quoted.
- "תמלל את הסרטון הזה", "מה נאמר ב…", "transcribe", "summarize this video".
- A local audio/video file needs transcription (pass the file path instead of a URL).

## Workflow

### 1. One-time setup (only if not yet installed)
Check whether the venv exists: `ls ~/.claude/skills/youtube-to-text/.venv/bin/python`.
If it does **not** exist, run setup once (installs `yt-dlp`, `ffmpeg`, `uv`, and builds an
isolated Python 3.12 venv with `faster-whisper`):

```bash
bash ~/.claude/skills/youtube-to-text/scripts/setup.sh
```

This is heavy (Homebrew installs + venv). Run it in the background if it's slow and tell the
user it's installing.

### 2. Transcribe
Run the transcriber with the skill's venv python. **Default language is Hebrew** — pass
`--lang en` for English-spoken videos. Pick the language from what the user says or the video;
when unsure, ask.

```bash
# Local (default) — free/offline:
~/.claude/skills/youtube-to-text/.venv/bin/python \
  ~/.claude/skills/youtube-to-text/scripts/transcribe.py \
  "<YOUTUBE_URL_OR_FILE>" --lang he

# ElevenLabs Scribe v2 — best Hebrew accuracy (needs an API key, see below):
~/.claude/skills/youtube-to-text/.venv/bin/python \
  ~/.claude/skills/youtube-to-text/scripts/transcribe.py \
  "<YOUTUBE_URL_OR_FILE>" --lang he --engine elevenlabs
```

The script prints progress to stderr and the final **transcript path on stdout** (capture it).
By default it writes to `./transcripts/<title>.txt` in the current working directory.

Useful flags:
- `--engine local|elevenlabs` — transcription engine (default `local`).
- `--lang he|en` — spoken language (default `he`). Local: Hebrew→ivrit.ai, English→Whisper large-v3.
  For ElevenLabs it's an accuracy hint passed to Scribe.
- `--accurate` — [local] use the full `large-v3` instead of `turbo` (slower, a bit more accurate).
- `--diarize` — [elevenlabs] label speakers (`**Speaker 1:** …`) in the output.
- `--out <path>` — output `.txt` path, or a directory.
- `--srt` — also write a timestamped `.srt` subtitle file.
- `--keep-audio` — keep the intermediate 16 kHz WAV.

**Notes**
- **Local:** first run downloads the model from HuggingFace (~1.6 GB for turbo). CTranslate2 on
  Mac runs on **CPU** (no Metal GPU), so a long video takes a few minutes — turbo (the default)
  is the practical choice. Long jobs: run in the background. The ivrit.ai models are
  **Hebrew-only by design**; don't use `--lang he` on English audio with the local engine.
- **ElevenLabs:** needs an API key (see *ElevenLabs setup* below). It costs money and uploads
  the audio to ElevenLabs — fine for your own videos; ask before sending anything sensitive.

#### ElevenLabs setup (API key)
The script looks for the key in this order: **(1)** `ELEVENLABS_API_KEY` env var →
**(2)** macOS **Keychain** (service `ELEVENLABS_API_KEY`, account `youtube-to-text`) →
**(3)** an `.env` file at `~/.claude/skills/youtube-to-text/.env`. **The user stores the key
themselves** (Claude never handles the raw secret). Recommended = Keychain (no plaintext on
disk); the store command prompts for the secret so it never lands in shell history:

```bash
security add-generic-password -s ELEVENLABS_API_KEY -a youtube-to-text -w
# (paste the key when prompted; -U to update an existing entry)
```

Alternative (simpler, plaintext but `chmod 600`): create `~/.claude/skills/youtube-to-text/.env`
with `ELEVENLABS_API_KEY=...`. If the key isn't found, the script exits with a clear message.

### 3. Proofread — **Hebrew only, mainly for `--engine local`** (Claude does this)
The **local** Hebrew model is excellent on the Hebrew itself but reliably garbles **English
technical/brand terms spoken inside Hebrew speech** (it spells them phonetically). After a
local `--lang he` transcription, **read the `.txt` and fix the clear errors**. (ElevenLabs
Scribe v2 usually gets these right, so this step is mostly unneeded with `--engine elevenlabs` —
still skim it.):

1. First preserve the raw model output: `cp "<title>.txt" "<title>.raw.txt"`.
2. Then edit `<title>.txt` in place, fixing only **unambiguous** mistakes — using context to
   resolve them. Typical fixes (the same term often appears mangled several different ways):
   - **Brand/tool names** → e.g. `CloudCode`/`iCloud`/`glוד` → **Claude Code**;
     `קלאופר`/`קלאודפלר`/`קלרוף` → **Cloudflare**; `רנגלר`/`באנגלג` → **Wrangler**;
     `ג'מיני` → **Gemini**; `ווב טריפורמס`/`Web3Fורמס` → **Web3Forms**;
     `paste-peat inside` → **PageSpeed Insights**; spell mangled domains/URLs back correctly.
   - **Garbled loanwords** → `פרמוורק`/`פרימוורק`/`פרמורג` → **framework** (or `פריימוורק`).
   - **Obvious single-word slips** where context makes the right word unambiguous
     (e.g. `ייצוב` → `עיצוב`).
3. **Rules — stay conservative:** fix only what is *clearly* wrong. Do **not** paraphrase,
   "improve", restructure, or change the speaker's wording/meaning. When a fix is genuinely
   ambiguous, leave it and (optionally) note it. Keep the same paragraph breaks.
4. Briefly tell the user what you corrected (a short list is enough).

(English transcripts — `--lang en` — generally don't need this; skip it unless something is
obviously wrong.)

### 4. Summarize (Claude does this — it is part of the skill)
After proofreading, **read the (corrected) `.txt`** and produce a concise summary in the user's
language (Hebrew for Hebrew videos). Write the summary to `<title>.summary.md` next to the
transcript **and** show the key points in chat. A good summary includes: a 1–2 sentence TL;DR,
the main points/sections as bullets, and any concrete takeaways, names, numbers, or links
mentioned.

## Output
- `transcripts/<title>.txt` — clean, paragraph-segmented transcript (proofread, for Hebrew).
- `transcripts/<title>.raw.txt` — raw ASR output before proofreading (Hebrew only).
- `transcripts/<title>.summary.md` — Claude's summary.
- (optional) `transcripts/<title>.srt` — subtitles, if `--srt` was passed.

## Not yet (easy to add later)
- Local speaker diarization (the `local` engine has none — diarization is ElevenLabs-only for
  now; would need `pyannote` to add it on-device).
- An OpenAI Whisper cloud engine behind `--engine` (the engine layer is already structured for it).

## Files
- `scripts/transcribe.py` — the pipeline (yt-dlp → ffmpeg → local/ElevenLabs → txt/srt).
- `scripts/setup.sh` — one-time installer (yt-dlp, ffmpeg, uv, venv + faster-whisper + requests).
