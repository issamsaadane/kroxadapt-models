# llm.gemma-4-e4b-it

- **Family:** llm
- **Task:** instruction-tuned text generation for short-form creative copy
- **Runtime:** TBD (most likely `llama.cpp` via GGUF; alternatives: ONNX Runtime GenAI, MLX on Apple Silicon)
- **Filename:** TBD (likely `gemma-4-e4b-it-Q4_K_M.gguf` or similar quantization)
- **Release:** not yet (planned)
- **Required for V1:** no
- **Bundled by default:** no

> **Status: roadmap entry, no asset yet.** This model is not in `models-manifest.json` until the runtime is locked, a vetted GGUF (or alternative) exists, and its SHA-256 is known.

## What it would do

Gemma 4 E4B-IT (4B-parameter "Efficient" instruction-tuned variant) is Google's small, on-device LLM aimed at desktops and laptops. KroxAdapt Studio will use it for **local creative copy operations** where round-tripping to a cloud LLM is overkill, slow, expensive, or privacy-sensitive.

## Product feature

**Brand localization + creative copy** — powers:
- **Subtitle shortening** — collapse a verbose Whisper transcript line so it fits the on-screen reading rate.
- **CTA rewrite** — adapt a call-to-action to platform-specific tone (short for TikTok, slightly longer for YouTube end cards).
- **Layout-aware copy fitting** — given a fixed text box and a target string, produce a variant that fits without truncation.
- **Brand-voice nudging** — light rewriting toward a brand style guide stored in the project.

## Why this model

- **4B class** — large enough for useful rewrites; small enough to run quantized on a creator laptop.
- **Instruction-tuned** — handles short imperative prompts well ("shorten this to 6 words and keep the CTA").
- **E (Efficient) variant** — better tokens-per-second on CPU than dense 4B competitors.
- **Local** — proprietary brand copy and unreleased creative never leave the machine.

## Why "TBD" runtime

Gemma 4 E4B-IT is shipped by Google in checkpoint formats meant for cloud / GPU inference. To run it inside a Tauri desktop app we need:
- A CPU-friendly quantization (typically Q4_K_M or Q5_K_M in GGUF).
- A runtime with stable Rust bindings (`llama.cpp` via `llama-cpp-rs` is the strongest candidate today).
- Acceptable tokens-per-second on a baseline machine (target: ≥ 8 tok/s on a 4-year-old laptop for the small rewriting tasks we do).

Candidates:
- **`llama.cpp` + GGUF** — most likely. Mature, cross-platform, well-tested with Gemma family.
- **ONNX Runtime GenAI** — possible if a clean ONNX export exists with acceptable quant.
- **MLX (Apple Silicon only)** — strong perf on M-series, but Mac-only, so it would have to coexist with another runtime for Windows.

The runtime choice determines the binary format and the manifest entry. Locking this is V1.x work.

## Provenance (when shipped)

Will likely originate from:
- The official Google release on Hugging Face (`google/gemma-4-e4b-it` or equivalent), or
- A community quantized GGUF derivative (subject to vetting and license confirmation).

License compliance review is required before shipping any quantized variant as a re-hosted asset. Provenance and license terms will be locked in this doc and in `models-manifest.json` when the asset ships.

## Fallback

When the local LLM is unavailable or not yet shipped, Studio's brand-copy step falls back to:
1. **Cloud LLM** (if user has credit and the project allows off-device LLM calls), or
2. **Manual copy** — Studio shows the source string and a target constraint (max chars, target tone), no AI assist.

## Roadmap dependencies

- Depends on runtime decision (`llama.cpp` strongly preferred for cross-platform parity).
- Depends on Gemma 4 license review for redistribution as a release asset.
- Depends on a benchmark pass on baseline creator hardware.
- Depends on prompt-template stability — Studio will rely on a small, versioned prompt library; if Gemma's templating changes between revisions we pin a specific instruct variant.

## Surface in app

The model should appear in the UI as "Local copy assist" or feature-specific names like "Shorten subtitle", "Fit to layout", "Adapt CTA" — never "Gemma." The string "gemma" or model implementation details should only appear in:
- About dialog
- Debug / developer mode logs
- Model manager screen (advanced)
