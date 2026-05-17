# speech.whisper

KroxAdapt Studio ships **Whisper** for local, offline transcription of audio and video. Multiple size variants are available; all run through `whisper.cpp` on the user's CPU.

| Model ID | Filename | Size | Required V1 | Bundled | Release |
|---|---|---:|---|---|---|
| `speech.whisper-base` | `ggml-base.bin` | ~148 MB | no | no | v1.0.0 |
| `speech.whisper-small` | `ggml-small.bin` | ~488 MB | no | no | v1.0.0 |
| `speech.whisper-medium` | `ggml-medium.bin` | ~1.5 GB | no | no | v1.1.0 |

None are bundled with the installer — all are **opt-in downloads** from the release assets on this repo.

## Product feature

**Transcription** — auto-captions, searchable timeline audio, text input for downstream creative tools (titles, summaries, translation).

## What it does

Whisper transcribes spoken audio into text with timestamps. KroxAdapt Studio uses it to:
- Auto-generate captions for video projects.
- Power search across timeline audio.
- Feed text into the translation pipeline (`translation.nllb-200-distilled-600m`).
- Feed text into the local LLM for subtitle shortening / CTA rewrite (`llm.gemma-4-e4b-it`).

## Which variant to pick

- **`base` (~148 MB)** — the lightweight option. Fastest, lowest memory ceiling, slightly less accurate. Picked automatically when the host machine is constrained (older laptops, low free RAM) or when the user explicitly opts for speed.
- **`small` (~488 MB)** — recommended balance for V1/V2. Better accuracy, especially for non-native English and noisier audio. Default for most modern hardware.
- **`medium` (~1.5 GB)** — high-accuracy variant. Strongest performance on multilingual content, noisy audio, accented speech, and overlapping speakers. Recommended for **Pro tier** workflows where transcript quality matters more than runtime. Significantly slower per minute of audio than `small`. Requires more RAM and more disk.

The Studio's model manager surfaces this as a single "Local transcription" feature; the picker between base / small / medium happens in advanced settings or via hardware-aware automatic selection.

## Provenance

All three `.bin` files are GGML format, mirrored from [`ggerganov/whisper.cpp`](https://huggingface.co/ggerganov/whisper.cpp) on Hugging Face. We re-host them here so the desktop app never depends on a third-party CDN at runtime, and so the manifest's SHA-256 is the integrity contract — not an upstream URL that could change.

## How KroxAdapt Studio uses it

1. User enables local transcription for the first time → Studio downloads the chosen variant from the matching release on this repo (`v1.0.0` for base/small, `v1.1.0` for medium).
2. Binary is validated against `models-manifest.json` `sha256`.
3. `whisper.cpp` runs in-process; audio is streamed off the timeline; segments are produced with timestamps.
4. Result is committed to the project as caption tracks or searchable transcript.

## Fallback

If no local model is installed and the user requests transcription, Studio falls back to:
1. **Cloud transcription** (if the user is signed in and has credit), or
2. **Manual transcription mode** with no automated suggestions.

The fallback is explicit in the UI — the user is told that local transcription is unavailable and what to do about it.

## Future

- May add `whisper-large-v3` (~3 GB) for users wanting maximum accuracy on top hardware.
- May add domain-fine-tuned variants (e.g. broadcast, podcast).
- May expose multi-language vs English-only variants separately.
- Subtitle alignment / forced alignment pass is a candidate post-V1.

## Surface in app

This model should appear in the UI as "Local transcription". The string "Whisper" should only appear in:
- About dialog
- Debug / developer mode logs
- Model manager screen (advanced)

The size variant picker (base / small / medium) should be described by user-facing value, not raw size:
- base → "Fast (lower accuracy)"
- small → "Recommended"
- medium → "Pro / multilingual"
