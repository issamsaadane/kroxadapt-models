# vision.yolo11n

- **Family:** vision
- **Task:** object-detection
- **Runtime:** onnxruntime
- **Filename:** `yolo11n.onnx`
- **Release:** `v1.0.0`
- **Required for V1:** yes
- **Bundled by default:** yes

## What it does

YOLOv11-nano is the smallest model in the YOLOv11 family. KroxAdapt Studio uses it for **local, real-time subject detection** on images and video frames. The detected bounding boxes feed the auto-cropping system that suggests subject-aware reframing for vertical / square / landscape exports.

Inference runs on the user's CPU via `onnxruntime` — no GPU required, no network round-trip.

## Why this model

- **Tiny footprint** (~11 MB) — bundles cleanly with the installer.
- **CPU-friendly** — runs in milliseconds per frame on modern laptops.
- **General COCO classes** — covers the common subjects (person, animal, vehicle, food, etc.) that creative editors actually crop around.

## Source

The `.onnx` weights ship here as a release asset and are mirrored from the [`Xenova/yolov11`](https://huggingface.co/Xenova/yolov11) ONNX export on Hugging Face. We re-host them so the desktop app never depends on a third-party CDN at runtime.

## How KroxAdapt Studio uses it

1. On install, the bundled `yolo11n.onnx` is validated against `models-manifest.json[vision.yolo11n].sha256`.
2. If missing or corrupt → download from the v1.0.0 release on this repo.
3. On user action (e.g. "Auto-frame for Instagram Reels"), Studio runs detection per keyframe, picks the dominant subject, and proposes a centered crop.

## Fallback

If the model is unavailable for any reason (file missing, runtime failure, sandbox restriction), Studio falls back to a **center-cropped** rectangle. No cloud call, no error popup — just a slightly less smart crop.

## Future

- V2 may add larger YOLOv11 variants (`yolo11s`, `yolo11m`) as optional upgrades for users on stronger hardware.
- May add segmentation variants (`yolo11n-seg`) for cleaner foreground/background separation.

## Surface in app

This model should appear in the UI as "Auto-frame subject" or similar. The string "YOLO" should only appear in:
- About dialog
- Debug / developer mode logs
- Model manager screen (advanced)
