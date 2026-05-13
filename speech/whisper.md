# speech.whisper

KroxAdapt Studio ships two Whisper variants for **local, offline transcription** of audio and video. Both run through `whisper.cpp` on the user's CPU.

| Model ID | Filename | Size | Required V1 | Bundled |
|---|---|---|---|---|
| `speech.whisper-base` | `ggml-base.bin` | ~147 MB | no | no |
| `speech.whisper-small` | `ggml-small.bin` | ~466 MB | no | no |

Neither is bundled with the installer — both are **opt-in downloads** from the v1.0.0 release on this repo.

## What it does

Whisper transcribes spoken audio into text. KroxAdapt Studio uses it to:
- Auto-generate captions for video projects.
- Power search across timeline audio.
- Feed text input into downstream creative tools (titles, summaries, etc.).

## Why two sizes

- **`base`** is the lightweight option. Faster, lower memory ceiling, slightly less accurate. Picked automatically when the host machine is constrained (older laptops, low free RAM) or when the user explicitly opts for speed.
- **`small`** is the recommended balance for V1/V2. Better accuracy, especially for non-native English and noisier audio. Picked by default when the machine can handle it.

The Studio's model manager surfaces this as a single "Local transcription" feature; the picker between base/small happens in advanced settings or automatically.

## Source

Both `.bin` files are GGML format and ship here as release assets, mirrored from the [`ggerganov/whisper.cpp`](https://huggingface.co/ggerganov/whisper.cpp) Hugging Face repository.

## How KroxAdapt Studio uses it

1. User enables local transcription for the first time → Studio downloads the chosen variant from the v1.0.0 release.
2. Binary is validated against `models-manifest.json` `sha256`.
3. `whisper.cpp` runs in-process; audio is streamed off the timeline; segments are produced with timestamps.
4. Result is committed to the project as caption tracks or searchable transcript.

## Fallback

If neither local model is installed and the user requests transcription, Studio falls back to:
1. **Cloud transcription** (if the user is signed in and has credit), or
2. **Manual transcription mode** with no automated suggestions.

The fallback is explicit in the UI — the user is told that local transcription is unavailable and what to do about it.

## Future

- V2 may add `whisper-medium` (~1.5 GB) for users wanting maximum accuracy.
- May add domain-fine-tuned variants (e.g. broadcast, podcast).
- May expose multi-language vs English-only variants separately.

## Surface in app

This model should appear in the UI as "Local transcription". The string "Whisper" should only appear in:
- About dialog
- Debug / developer mode logs
- Model manager screen (advanced)
