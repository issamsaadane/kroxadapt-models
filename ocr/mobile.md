# ocr.mobile

- **Family:** ocr
- **Task:** detect + recognize burned-in / on-screen text
- **Runtime:** onnxruntime (planned)
- **Filenames:** TBD — typically an OCR pipeline ships as a **detector + recognizer pair** (e.g. `det.onnx` + `rec.onnx`), not a single file. Final names will be locked at first release.
- **Release:** not yet (planned)
- **Required for V1:** no
- **Bundled by default:** no

> **Status: roadmap entry, no asset yet.** This model is not in `models-manifest.json` until the file set is finalized and SHA-256 is known for each asset.

## What it would do

A small, mobile-class OCR pipeline that reads text directly off video frames and stills: burned-in subtitles, supers, lower-thirds, disclaimers, end cards, watermarks. KroxAdapt Studio uses these reads for **creative QA** and to surface "secret" text that exists in the pixels but isn't in any timeline asset.

## Product feature

**Creative QA + layout safety** — powers:
- **Untranslated-text detection** — flag burned-in copy that wasn't translated when shipping a localized cut.
- **Text / logo overlap QA** — confirm overlays don't sit on top of existing supers or burned-in disclaimers.
- **Disclaimer presence checks** — required-text QA (e.g. a regulated industry disclaimer that must appear in every cut).
- **End-card OCR** — extract CTAs / copy that already exists on the video so brand-localization rewrites stay consistent.

## Why a "mobile" OCR

- **Performance budget** — OCR is run on many frames (not just keyframes), so per-frame cost must be low.
- **CPU-only target** — runs on the same `onnxruntime` already shipped for the YOLO models.
- **Detector + recognizer separation** — the detector runs first and is cheap; the recognizer only runs on detected text boxes, which is where the heavy work lives.

## Provenance (when shipped)

Candidate pipelines (one will be picked after benchmarking):
- **PaddleOCR mobile** ONNX exports (PP-OCRv4 mobile or v5 mobile) — strong multilingual support, well-known ONNX builds available.
- **docTR** small ONNX variants — Apache-2 licensed, easier license story.
- **Tesseract + a learned detector** — battle-tested but slower; only a fallback choice.

Vetting requirements before shipping:
- License is compatible with redistribution as a release asset.
- ONNX files run on `onnxruntime` CPU with Mac M-series + Win x86_64 parity.
- Accuracy passes an internal eval on broadcast / social-media frame types.

Provenance and exact file names will be locked in this doc and in `models-manifest.json` when the assets ship.

## How KroxAdapt Studio will use it

1. User triggers QA pass (or it runs automatically on export readiness check).
2. Studio walks selected frames — keyframes + scene-change boundaries — and runs detector then recognizer.
3. Detected text strings + timestamps + bounding boxes feed:
   - Untranslated-text flag list, with a thumbnail and timecode for each.
   - Overlap checks against the project's overlay tracks.
   - Required-text presence/absence list, surfaced in the export-readiness panel.

## Fallback

When OCR is unavailable, the QA pass falls back to:
1. **Manual review** — Studio shows a frame-strip of likely-text frames and asks the user to confirm presence / translation status.
2. **Cloud OCR** for users who have credit and accept off-device frame processing.

## Roadmap dependencies

- Depends on picking a final detector + recognizer pair.
- Depends on language coverage decisions (CJK and RTL require extra recognizer work; V1 may ship Latin-only).
- Depends on a confidence-threshold benchmark so the QA flag isn't noisy.

## Surface in app

The pipeline should appear in the UI as "Text in frame" or feature-specific names like "Find untranslated text", "Disclaimer check" — never "OCR engine X." Implementation strings ("paddle", "doctr", "rec", "det") should only appear in:
- About dialog
- Debug / developer mode logs
- Model manager screen (advanced)
