# MLR exercise 8 — Live leaf classifier (web)

**Open the app:** <https://petrakasparova.github.io/mlr-ex08/>

Classifies leaves into **apple / cherry / chestnut / maple** directly in the browser
from the phone's rear camera, using `leaf_model.onnx` with
[onnxruntime-web](https://cdn.jsdelivr.net/npm/onnxruntime-web@1.21.0/dist/ort.min.js)
(WASM). No server-side inference — everything runs on the device.

## Usage

1. Open the URL above on a phone (or desktop) — the model (~6 MB) loads first.
2. Tap **▶ Start rear camera** and allow camera access. All four class
   probabilities update live (~4 inferences/s).
3. If the camera is unavailable, use **📷 Upload photo** as a fallback.

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

## Files

| file | role |
|---|---|
| `index.html` | the whole app (plain JavaScript, no build step) |
| `leaf_model.onnx` | the trained leaf classifier |
| `model_info.json` | classes, input size, normalization constants (fetched at runtime) |
| `qrcode-generator.js` | MIT-licensed [qrcode-generator](https://github.com/kazuhikoarase/qrcode-generator) v1.4.4 — the QR code at the bottom of the page is rendered **in the browser**, no external QR service |
| `check_image.jpg`, `check_expected.json` | reference image + expected probabilities |

## Requirements

A browser with WebAssembly SIMD and `getUserMedia` over HTTPS — e.g. **iOS
Safari 16.4+** (iPhone), recent Chrome/Firefox/Edge. The page works on the
phone's rear camera via `facingMode: environment` and `<video playsinline>`.
