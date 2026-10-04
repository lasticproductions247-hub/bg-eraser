# BG Eraser — 100% On-Device Background Remover

**Real AI background removal that runs entirely in your browser.**
ONNX Runtime Web + transformers.js. No upload, no account, no API key, no server.

🔗 **Live:** https://lasticproductions247-hub.github.io/bg-eraser/

---

## Why this isn't hosted on Hugging Face

The original version of this app called the Hugging Face serverless Inference API.
**That endpoint no longer exists.** `api-inference.huggingface.co` was fully
decommissioned (it now fails DNS resolution outright), and the replacement
`router.huggingface.co/hf-inference` returns `401 Unauthorized` without a
bearer token — which would mean shipping an API key inside a public web page.
Both are dead ends for a keyless static site.

So this version runs the model **on your machine**:

| | Old (broken) | This version |
|---|---|---|
| Where it runs | Hugging Face servers | Your browser, via WebAssembly |
| Needs an API key | No — but the API is gone | No |
| Your image | Uploaded to HF | **Never leaves the device** |
| Works offline | No | Yes, after first load |
| Cost per image | Rate-limited | Free, unlimited |

## How it works

1. `transformers.js` is imported from a CDN and the ONNX model is fetched from
   the Hugging Face Hub — both public, no token required.
2. `pipeline('background-removal', …)` runs the segmentation model in WebAssembly.
3. The resulting mask is written into the alpha channel of your original pixels,
   composited on a `<canvas>`, and encoded to a PNG — entirely in the page.
4. The browser caches the weights, so every image after the first is instant.

### Models

| Model | Size | Character |
|---|---|---|
| **MODNet** (default) | 6.6 MB | Fast, good on most subjects |
| **RMBG-1.4** | 44 MB | Sharper edges, better on hair/fur |

Measured from the actual `onnx/model_quantized.onnx` weights each repo's config
selects (dtype `q8`); both were downloaded and confirmed to be valid ONNX.

Both are grabbed straight from the Hub and cached by the browser.

## Features

- Drag & drop, click to browse, or `Ctrl+V` to paste
- Live before/after comparison slider (mouse + touch)
- Real per-file download progress during the model download
- Cancel button during processing
- Result validation — a blob that won't decode is reported, not silently saved
- Filenames set via `textContent`, never `innerHTML` (no markup injection)
- Full keyboard access on the dropzone, `aria` labelling, `prefers-reduced-motion`
- Object URLs revoked on reset so memory doesn't leak across images

## Tech

Single file, no build step.

- Vanilla ES modules
- [transformers.js](https://github.com/huggingface/transformers.js) 3.7.6
- ONNX Runtime Web (bundled inside transformers.js)
- Tailwind CDN + Font Awesome 6.5.1
- Space Grotesk

## Run locally

It's static — but ES modules and WASM need a real origin, so use a server
rather than double-clicking the file:

```bash
python -m http.server 8000
# then open http://localhost:8000
```

## Privacy

No upload endpoint exists in this codebase. No analytics, no telemetry. The
only network requests are to `cdn.jsdelivr.net` (library) and
`huggingface.co` (model weights) — both fetch code and weights, never your
image. After the model is cached, you can disconnect entirely and it still works.

## Licence

MIT.