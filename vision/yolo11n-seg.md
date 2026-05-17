# vision.yolo11n-seg

- **Family:** vision
- **Task:** instance-segmentation
- **Runtime:** onnxruntime
- **Filename:** `yolo11n-seg.onnx`
- **Release:** `v1.1.0`
- **Required for V1:** no
- **Bundled by default:** no

## What it does

YOLOv11n-seg adds **per-pixel instance masks** on top of YOLOv11n's bounding-box detection. Instead of just "there is a person here, in this rectangle," it produces the exact silhouette of the person within that rectangle. KroxAdapt Studio uses these masks for layout safety, logo / text placement, and overlap QA.

Same runtime as `vision.yolo11n` — CPU `onnxruntime`, no GPU required, no network round-trip.

## Product feature

**Layout safety + creative QA** — powers:
- Safe-zone calculation for logos, supers, and end-card text so they don't sit on top of the subject.
- Subject vs background separation cues for the auto-cropper (already shipping with `vision.yolo11n`, but with masks the crop can avoid clipping the silhouette).
- QA checks: "Does the watermark overlap a person's face? Does the CTA cover the product?"

## Why this model

- **Same small footprint** as the detection variant (~11 MB), same CPU performance profile.
- **Same COCO class set** as `yolo11n`, so the detection and segmentation pipelines stay in sync.
- **Layout-safety masks are cheaper than full alpha matting** — perfect for "good enough" decisions like nudging an overlay or checking for occlusion.

## Provenance

The `.onnx` weights ship as a v1.1.0 release asset, **exported locally from the official `yolo11n-seg.pt`** at [`Ultralytics/YOLO11`](https://huggingface.co/Ultralytics/YOLO11), using:

```
ultralytics==8.4.49
yolo export model=yolo11n-seg.pt format=onnx opset=12 simplify=False
```

Same export pipeline as `yolo11n.onnx` — reproducible, no third-party CDN dependency at runtime.

## How KroxAdapt Studio uses it

1. User enables advanced layout safety / mask-aware features → Studio downloads `yolo11n-seg.onnx` from v1.1.0 on demand.
2. Binary is validated against `models-manifest.json[vision.yolo11n-seg].sha256`.
3. Studio runs segmentation on selected frames (typically keyframes / hero frames, not every frame) to produce instance masks.
4. Masks feed:
   - Safe-zone polygons for overlay placement.
   - Overlap detection between subjects and creative elements (CTA, logo, captions).
   - Optional smarter crop suggestions that respect silhouette boundaries.

## Fallback

If the segmentation model is unavailable, Studio falls back to **bounding-box-only layout safety** (using `vision.yolo11n`). Overlay placement still avoids subject rectangles, but won't account for negative space inside the bounding box (e.g. the gap between someone's arm and torso).

## Future

- Larger seg variants (`yolo11s-seg`, `yolo11m-seg`) as optional upgrades for users with stronger hardware.
- Custom-trained segmentation for brand-specific assets (e.g. product packaging classes) — post-V1.

## Surface in app

The model should appear in the UI as "Smart layout safety" or "Subject masks" — never "YOLO seg." The string "yolo11n-seg" should only appear in:
- About dialog
- Debug / developer mode logs
- Model manager screen (advanced)
