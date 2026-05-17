# translation.nllb-200-distilled-600m

- **Family:** translation
- **Task:** multilingual machine translation
- **Runtime:** TBD (candidates: CTranslate2, ONNX Runtime, llama.cpp-style native loader)
- **Filename:** TBD (depends on runtime — `.bin`, `.ct2/`, `.onnx`, etc.)
- **Release:** not yet (planned)
- **Required for V1:** no
- **Bundled by default:** no

> **Status: roadmap entry, no asset yet.** This model is not in `models-manifest.json` until the runtime is locked, a vetted binary exists, and its SHA-256 is known.

## What it would do

NLLB ("No Language Left Behind") 200 distilled 600M is Meta's compact multilingual translation model — 200 languages covered in a single ~1.2 GB checkpoint. KroxAdapt Studio will use it for **local, offline subtitle / copy translation** without round-tripping caption text to a cloud API.

Direct path: Whisper transcript → NLLB → translated subtitle track. No external service in the loop.

## Product feature

**Translation** — auto-translation of:
- Generated subtitles (from `speech.whisper-*`).
- Burned-in supers / overlays selected via the OCR pipeline (`ocr.mobile`).
- Manually written copy that needs to ship in multiple markets.

## Why this model

- **200 languages**, including many low-resource ones that cloud APIs handle poorly or charge premium for.
- **600M distilled** — small enough to run on a creator laptop without a discrete GPU.
- **Single checkpoint** — no per-language-pair models to manage.
- **Offline** — caption text never leaves the machine, important for unreleased / NDA work.

## Why "TBD" runtime

The original Meta checkpoint is PyTorch. To ship inside a Tauri desktop app we need a runtime that:
- Does not require shipping PyTorch (huge).
- Runs on CPU on Mac (Apple Silicon) and Windows (x86_64) with reasonable speed.
- Has a stable C / Rust binding usable from Tauri.

Candidates being evaluated:
- **CTranslate2** — fast, well-supported, has NLLB conversions on Hugging Face. Likely first choice.
- **ONNX Runtime** — works, but NLLB ONNX exports are non-trivial and not all variants are pre-built.
- **Native llama.cpp-style port** — would require porting work, not free.

The runtime choice determines the binary format and the manifest entry. Locking this is V1.x work.

## Provenance (when shipped)

Will likely originate from one of:
- The official Meta release on Hugging Face (`facebook/nllb-200-distilled-600M`).
- A CTranslate2 conversion of the above (e.g. `JustFrederik/nllb-200-distilled-600M-ct2` family).

Provenance will be locked in this doc and in `models-manifest.json` when the asset ships.

## Fallback

When NLLB is unavailable or not yet shipped, Studio's translation step falls back to:
1. **Cloud translation** (if user has credit and consents to send copy off-device), or
2. **Manual translation** — Studio surfaces the source strings and lets the user paste translations.

## Roadmap dependencies

- Depends on runtime decision (CTranslate2 vs ONNX vs other).
- Depends on a benchmark pass on Mac M-series + Intel Win11 baseline machines.
- Depends on confirming model file size + RAM ceiling on typical creator hardware.

## Surface in app

The model should appear in the UI as "Local translation" — never "NLLB." The string "nllb" or model implementation details should only appear in:
- About dialog
- Debug / developer mode logs
- Model manager screen (advanced)
