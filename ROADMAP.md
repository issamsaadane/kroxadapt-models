# KroxAdapt Models Roadmap

This document tracks every model in the registry — shipped and planned — and maps each to the product feature it powers in KroxAdapt Studio.

The **manifest** (`models-manifest.json`) is the source of truth for *what ships*: it only contains entries with a vetted binary, a real `sha256`, and a real `sizeBytes`. This document is the source of truth for *what's coming*.

## Product features → models

| Product feature | Model(s) |
|---|---|
| Framing / scene intelligence | `vision.yolo11n` *(shipped v1.0.0)* |
| Transcription | `speech.whisper-base` *(shipped v1.0.0)*, `speech.whisper-small` *(shipped v1.0.0)*, `speech.whisper-medium` *(shipping v1.1.0)* |
| Translation | `translation.nllb-200-distilled-600m` *(planned)* |
| Brand localization | `llm.gemma-4-e4b-it` *(planned)* |
| Layout safety | `vision.yolo11n-seg` *(shipping v1.1.0)* |
| Creative QA | `ocr.mobile` *(planned)*, `vision.yolo11n-seg` *(shipping v1.1.0)* |
| Advanced scene detection | `vision.omnishotcut` *(planned)* |

Every model in the registry maps to at least one product feature. If a model doesn't have a feature, it doesn't ship.

## Priority order

In rough order of expected ship sequence after v1.0.0:

1. **`speech.whisper-medium`** — pro-tier transcription. Same proven runtime (`whisper.cpp`) as base/small, same vetted source (`ggerganov/whisper.cpp`). **Lowest risk, highest user value.**
2. **`translation.nllb-200-distilled-600m`** — closes the localization loop with no cloud dependency. Runtime decision (CTranslate2 vs ONNX) is the gating item.
3. **`llm.gemma-4-e4b-it`** — local creative copy. Gates on license review and `llama.cpp` integration into Tauri.
4. **`vision.yolo11n-seg`** — layout-safety masks. Same Ultralytics export pipeline as `yolo11n`; easy to ship technically. Priority is below NLLB / Gemma because the bbox-only fallback already gives most of the layout-safety value.
5. **`ocr.mobile`** — burned-in text QA and overlap detection. Gates on picking a final detector + recognizer pair (PaddleOCR mobile vs docTR is the live decision).
6. **`vision.omnishotcut`** — advanced scene/transition detection. Most research-heavy of the set; ships when the ONNX export or a distilled replacement is validated.

## Shipped models — current state

| Model | Filename | Release | Runtime | Status |
|---|---|---|---|---|
| `vision.yolo11n` | `yolo11n.onnx` | v1.0.0 | onnxruntime | shipped |
| `speech.whisper-base` | `ggml-base.bin` | v1.0.0 | whisper.cpp | shipped |
| `speech.whisper-small` | `ggml-small.bin` | v1.0.0 | whisper.cpp | shipped |

## Planned models — status & blockers

| Model | Runtime | Blockers |
|---|---|---|
| `speech.whisper-medium` | whisper.cpp | none — same source as base/small |
| `vision.yolo11n-seg` | onnxruntime | none — same export pipeline as yolo11n |
| `translation.nllb-200-distilled-600m` | TBD (CTranslate2 candidate) | runtime decision; cross-platform benchmark |
| `llm.gemma-4-e4b-it` | TBD (`llama.cpp` candidate) | license review; runtime integration; prompt-template versioning |
| `ocr.mobile` | onnxruntime | detector+recognizer choice; language coverage; confidence-threshold tuning |
| `vision.omnishotcut` | TBD | license; ONNX export feasibility (or distillation); quality benchmark vs ffmpeg |

## Why some models stay "planned" without a manifest entry

Per the registry convention: a model only enters `models-manifest.json` when it has:
- A locked runtime that ships inside the Studio.
- A vetted asset file (or file set) at a known URL or local-export pipeline.
- A computed `sha256` and `sizeBytes` for every asset.

Until those exist, the model lives **here**, in this roadmap, and in its per-model doc under `vision/`, `speech/`, `translation/`, `llm/`, or `ocr/`.

## How models leave the roadmap

A model is removed from the planned list and added to the manifest when:
1. Its per-model doc has filled-in (not TBD) runtime, filename(s), and provenance.
2. A vetted asset has been uploaded to a release on this repo.
3. The manifest entry has a real `sha256` and `sizeBytes`.
4. KroxAdapt Studio has at least one feature wired to consume it.
