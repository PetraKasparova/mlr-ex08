# MLR exercise 8 — Live leaf classifier (web)

**Open the app:** <https://petrakasparova.github.io/mlr-ex08/>

Classifies leaves into **apple / cherry / chestnut / maple** directly in the browser
from the phone's rear camera, using `leaf_model.onnx` with ONNX Runtime Web 1.21.0
(WASM). No server-side inference — image processing and inference run entirely
locally in the browser; no image ever leaves the device.

## Usage

1. Open the URL above on a phone (or desktop) — the model (~6 MB) and runtime
   (~24 MB, cached after the first load) download first.
2. Tap **▶ Start rear camera** and allow camera access. All four class
   probabilities update live (~4 inferences/s).
3. If the camera is unavailable, use **📷 Upload photo** as a fallback.

If anything fails to load, the page reports the exact file, URL and HTTP status
under “Load diagnostics”, with a ↻ Retry button.

## Preprocessing (follows `model_info.json` exactly)

| step | detail |
|---|---|
| crop | central square of the camera frame |
| resize | to `input_size × input_size` = 96 × 96 |
| scale | RGB pixels ÷ 255 → 0–1 |
| normalize | `(px − mean) / std` per channel, mean `[0.485, 0.456, 0.406]`, std `[0.229, 0.224, 0.225]` |
| tensor | float32 NCHW, shape `1 × 3 × 96 × 96`, input name `image` |
| output | raw `logits` [1×4] → softmax → probabilities |

## Correctness check

`check_image.jpg` must reproduce the probabilities in `check_expected.json`.
The page has a built-in self-test — open
<https://petrakasparova.github.io/mlr-ex08/?selftest=1> — it classifies the
image through the exact same browser code path (canvas crop/resize + WASM
inference) and prints the per-class differences. Verified max absolute error:
**≈ 3 × 10⁻⁸**.

## Why the runtime is served from `vendor/` (same origin)

The first version loaded `onnxruntime-web` from the jsDelivr CDN. On some mobile
networks the CDN (or one of its WASM files) is unreachable (DNS filtering,
captive portals, content blockers), which surfaced only as a bare
“Failed to fetch”. The runtime files are therefore vendored into the
repository and served from the same GitHub Pages origin as the page itself —
no external requests at runtime. Pinned version: **onnxruntime-web 1.21.0**
(stable, needs only WASM SIMD + single thread, no SharedArrayBuffer/COOP/COEP).

## Files

| file | role |
|---|---|
| `index.html` | the whole app (plain JavaScript, no build step) |
| `leaf_model.onnx` | the trained leaf classifier |
| `model_info.json` | classes, input size, normalization constants (fetched at runtime) |
| `vendor/ort.min.js` + `vendor/ort-wasm-simd-threaded.jsep.*` | ONNX Runtime Web 1.21.0 (MIT) |
| `qrcode-generator.js` | MIT-licensed [qrcode-generator](https://github.com/kazuhikoarase/qrcode-generator) v1.4.4 — the QR code at the bottom of the page is rendered **in the browser**, no external QR service |
| `check_image.jpg`, `check_expected.json` | reference image + expected probabilities |

## Requirements

A browser with WebAssembly SIMD and `getUserMedia` over HTTPS — e.g. **iOS
Safari 16.4+** (iPhone), recent Chrome/Firefox/Edge on Android or desktop.
The rear camera is selected via `facingMode: environment` and `<video
playsinline>` (required by iOS Safari).
