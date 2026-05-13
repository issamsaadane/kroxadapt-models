# kroxadapt-models

Controlled model registry for **KroxAdapt Studio**.

## Purpose

KroxAdapt Studio uses this repository as the single, controlled source for local AI model metadata and binaries. The desktop app does not fetch from arbitrary upstream URLs (Ultralytics, Hugging Face, etc.) at runtime — it consults the manifest and release assets published here.

## How it works

- `models-manifest.json` is the source of truth: what models exist, which runtime they use, which are required for V1, which ship bundled, expected file size, sha256, fallback behavior.
- Actual model binaries (`.onnx`, `.bin`, `.gguf`, etc.) are **not committed to git history**. They live as **GitHub Release assets** on this repo.
- Releases are versioned (`v1.0.0`, `v1.1.0`, …). The manifest pins each model to a release.

## Distribution model

- **Required V1 models** are bundled inside the KroxAdapt Studio installer when possible. The release here is the *repair / re-download* source if the bundled file is missing, corrupt, or has been deleted by the user.
- **Optional models** (alternative speech models, future upgrades, larger variants) are downloaded on demand from the release assets when the user opts in.
- **Future model updates** ship as new releases. The Studio checks `models-manifest.json` to discover new versions.

All models in this registry run **locally / offline** after installation. No data leaves the user's machine for inference.

## UX rule for KroxAdapt Studio

The Studio UI should describe what a model *does* (e.g. "Auto-frame subject", "Local transcription"), not what library it uses. Implementation details like "YOLO" or "Whisper" should only surface in debug screens, the About panel, or developer-mode logs.

## Repo layout

```
.
├── README.md
├── models-manifest.json        # Source of truth, always pinned to a release
├── vision/
│   └── yolo11n.md              # Model-specific notes
└── speech/
    └── whisper.md              # Model-specific notes
```

Release assets (not in git):

```
v1.0.0
├── yolo11n.onnx
├── ggml-base.bin
└── ggml-small.bin
```

## For Studio engineers

1. On install, validate bundled binaries against the manifest's `sha256`.
2. If a required binary is missing or fails checksum, download from the matching release tag.
3. For optional models, download lazily from the release assets when the user enables the feature.
4. Never hardcode upstream URLs (e.g. `huggingface.co/...`) into the Studio. Always go through this manifest.

## Versioning

- `schemaVersion` — bumped only when the manifest shape changes.
- `registryVersion` — bumped on each new release of this repo.
- Per-model `release` field — pins each model to the GitHub Release tag where its binary lives.
