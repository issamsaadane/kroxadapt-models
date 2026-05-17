# vision.omnishotcut

- **Family:** vision
- **Task:** scene / shot / transition detection
- **Runtime today:** PyTorch (research checkpoint)
- **Runtime target for Studio:** TBD — likely ONNX Runtime after export, possibly a lighter learned re-implementation
- **Filename today:** `OmniShotCut_ckpt.pth`
- **Filename in Studio:** TBD after export
- **Release:** not yet (planned)
- **Required for V1:** no
- **Bundled by default:** no

> **Status: roadmap entry, no asset yet.** This model is not in `models-manifest.json` until the desktop runtime is locked, an export pipeline exists, and the resulting asset's SHA-256 is known.

## What it would do

OmniShotCut is a learned scene / shot / transition detector that handles the hard cases standard heuristics miss:
- **Jump cuts** within the same setting and lighting.
- **Subtle dissolves and fades** that don't trigger a pixel-difference threshold.
- **Wipes and graphical transitions.**
- **Non-cinematic sources** like anime, game capture, screen recordings — where the usual color-histogram approaches fail.

KroxAdapt Studio will use it as the **advanced scene-detection backend** for timeline-aware automations.

## Product feature

**Advanced scene detection** — powers:
- **Smart cut points** for auto-summary, highlight reels, social re-cuts.
- **Per-shot processing** — applying YOLO detection / OCR / framing analysis once per shot instead of once per frame, with the right scene boundary.
- **Transition-aware editing** — knowing the actual shot boundary in a fade or wipe instead of guessing.

## Why this model

Standard ffmpeg / OpenCV scene detection is brittle on:
- Animated / drawn content.
- Game capture with HUD overlays that mask underlying color shifts.
- Modern transitions (dissolves, slow pushes, match cuts).

A learned detector handles all of the above with one pipeline. The OmniShotCut research checkpoint is a strong candidate for the "advanced" tier; the "fast" tier stays on ffmpeg heuristics.

## Why "TBD" desktop runtime

The original `OmniShotCut_ckpt.pth` is a PyTorch checkpoint. Shipping it inside a Tauri app is not viable in PyTorch form — we don't want to ship PyTorch or run inference on a Python sidecar.

Path forward (any of these may end up being the answer):
- **Export to ONNX** — direct path if the model architecture is ONNX-friendly. Run via `onnxruntime`, same stack as YOLO + OCR.
- **Re-implement / distill** to a lighter learned classifier that takes per-frame embeddings and detects shot boundaries.
- **Hybrid** — keep ffmpeg heuristics as a prefilter and only call the learned model on ambiguous spans.

Locking the runtime + export is V1.x research-and-validation work, not a same-day task.

## Provenance (when shipped)

Original research checkpoint: `OmniShotCut_ckpt.pth`. License and redistribution rights must be confirmed before shipping any derived asset. Provenance will be locked in this doc and in `models-manifest.json` when the asset ships.

## How KroxAdapt Studio will use it

1. User triggers any pipeline that needs scene boundaries (highlight reel, auto-summary, per-shot color grade).
2. Studio decides between the **fast** (ffmpeg) and **advanced** (this model) backend based on source type and user setting.
3. The advanced backend produces a shot list with start/end timestamps and a confidence per boundary.
4. Downstream features (YOLO, OCR, transcription) align their per-frame work to those boundaries.

## Fallback

When the learned model is unavailable, Studio falls back to **ffmpeg-based scene detection** (color/histogram threshold tuned by source profile). Quality drops on animated / game-capture / soft-transition content, but the workflow still functions.

## Roadmap dependencies

- Depends on confirming license / redistribution.
- Depends on an ONNX export pass (or a distillation pass) that produces a runtime-viable artifact.
- Depends on benchmark vs ffmpeg on real KroxAdapt workloads to confirm the quality gap justifies the disk + CPU cost.

## Surface in app

The pipeline should appear in the UI as "Smart scene detection" or "Advanced scene mode." The string "OmniShotCut" should only appear in:
- About dialog
- Debug / developer mode logs
- Model manager screen (advanced)
