# BulkBG — Batch Background Removal

A browser-based tool for bulk AI background removal. Drop a folder of images, pick an output folder, and process hundreds of files in one go — no installation, no server, no data leaves your machine.

**Live app: [bulkbackgroundremover.netlify.app](https://bulkbackgroundremover.netlify.app/)**

---

## Features

- **Drag and drop** files or entire folders — mirrors the original folder structure in the output
- **AI background removal** powered by `@imgly/background-removal`, runs entirely in the browser
- **Model selector** — choose between speed and quality
- **Output format** — PNG (transparent) or WebP (smaller file size)
- **Retry on failure** — each file gets two attempts; failures are logged with the error
- **Live activity log** — timestamped, colour-coded, downloadable as `.txt`
- **Real-time stats** — queued, done, and failed counts in the header
- **No data uploaded** — all processing happens on-device

---

## Usage

1. Open the app in **Chrome** (other browsers not supported — see below)
2. Drop image files or folders into the drop zone, or click to browse
3. Choose a model and output format
4. Click **Choose output folder** and select where to save results
5. Click **▶ Start Processing**

Output files are saved as `nobg_originalname.png` (or `.webp`), preserving the original folder structure inside the output folder.

---

## Models

| Model | Speed | Quality | Notes |
|---|---|---|---|
| `isnet_fp16` | Fast | Good | Default — best balance |
| `isnet` | Slow | Best | Full precision |
| `isnet_quint8` | Fastest | Moderate | 8-bit quantised |

The model is downloaded once and cached by the browser. Subsequent runs load from cache.

---

## File Structure

```
BulkBgRemoval.html   — the entire app (single file)
_headers             — Netlify header config (enables WASM multithreading)
netlify.toml         — Netlify deploy config
```

---

## Why Netlify and not GitHub Pages?

The AI model uses WebAssembly (WASM) which can run on multiple CPU cores — but only if the page is served with two specific HTTP headers:

```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

GitHub Pages does not allow custom HTTP headers. Netlify does, via the `_headers` file. Without these headers the WASM runtime falls back to single-threading, which is slower and causes Chrome's "page unresponsive" popup during processing.

---

## Browser Support

**Chrome desktop only.** The app relies on:
- [File System Access API](https://developer.mozilla.org/en-US/docs/Web/API/File_System_Access_API) — for writing directly to a chosen output folder
- WebAssembly multithreading — for fast AI inference
- `webkitGetAsEntry` — for reading dropped folders recursively

Firefox and Safari do not support the File System Access API and cannot write files to disk from the browser.

---

## Local Development

Serve with cross-origin isolation headers enabled. A custom server script is included:

```bash
cd /path/to/project
python server.py
```

Then open `http://localhost:8080/BulkBgRemoval.html` in Chrome.

The standard `python -m http.server` will work but won't send the isolation headers, so WASM will run single-threaded and the unresponsive popup may appear.
