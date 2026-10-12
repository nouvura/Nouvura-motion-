# Nouvura Motion

A browser-based studio for animated explainer videos. Build scenes with text, shapes, images, SVG whiteboard drawings and programmatic infographics, add audio, preview in real time, and export an **MP4 (H.264 + AAC)** — all on your own device.

* **Private by design** — no account, no upload, no analytics. Projects and media are stored in your browser (IndexedDB).
* **Static site** — plain HTML/CSS/JavaScript, no build step, deployable to GitHub Pages as-is.
* **Deterministic rendering** — the live preview and the MP4 export use the same renderer, so what you see is what you export.

> © 2026 Nouvura. The logo in `assets/logo.svg` is a **temporary placeholder** — replace it with the official logo and set `logoIsPlaceholder: false` in `js/core.js`.

## Features

| Area | What is included |
| --- | --- |
| Scenes & timeline | Multiple scenes, drag-to-resize durations, drag/trim layer bars, ruler with zoom & fit, scrubbing, snapping, scene transitions (cut, fade, wipe, slide) |
| Layers | Text (multi-line, Arabic/Hebrew-aware RTL, outline, shadow, background box, animated counter), shapes (9 kinds, gradient, draw-on), icons (35), images, SVG |
| Animation | Entrance/exit effects (fade, slide, zoom, pop, wipe, draw-on, typewriter), 7 easings, per-property keyframes (position, scale, rotation, opacity) |
| Whiteboard | Real SVG path geometry is traced progressively (sequential / simultaneous, ordering modes, pen style, stylised drawing hand, colour fade-in). Raster reveals: wipe, radial, scribble, and a **Sobel edge-sketch → colour** reveal computed locally |
| Infographic studio | 15 data-driven kinds: bar, horizontal bar, line/area, pie, donut, progress, KPI counter, percentage ring, comparison, timeline, process, flowchart, steps, callout, legend; 5 style presets + your own saved presets; spreadsheet paste |
| Audio | MP3 / WAV / M4A import, waveform clips, trim, split, volume, fades, mixed down and encoded to AAC on export |
| Projects | Autosave, multiple projects, rename/duplicate/delete, portable `.nouvura.json` backup (media embedded) and re-import, storage usage |
| Editing | Undo/redo (120 steps), copy/paste, align, z-order, multi-select, keyboard shortcuts, lock/hide layers |
| Export | 1280×720, 1920×1080, 1080×1920, 1080×1080 · 24/25/30 fps · quality & speed presets · progress, ETA, cancel · playback verification |

## Run it

Serve the folder over HTTP (modules, workers and IndexedDB behave best from a real origin):

```bash
python3 -m http.server 8080      # or: npx serve .
# open http://localhost:8080
```

Opening `index.html` directly from disk (`file://`) is not a supported way to run it.

### Deploy to GitHub Pages

1. Push this folder to a repository.
2. In **Settings → Pages**, choose **GitHub Actions** (a ready workflow is in `.github/workflows/pages.yml`), or choose *Deploy from a branch* → `main` / root.
3. The app is then served at `https://<user>.github.io/<repo>/`. All paths are relative, so sub-folders work.

## MP4 export engine (FFmpeg WebAssembly)

Export uses [`@ffmpeg/ffmpeg`](https://github.com/ffmpegwasm/ffmpeg.wasm) 0.12 with the single-thread core, so **no special COOP/COEP headers are required** (works on GitHub Pages).

The engine files (~30 MB) are loaded in this order:

1. `vendor/ffmpeg/` in the same origin (fastest, works offline once the site is cached), then
2. jsDelivr CDN (`@ffmpeg/ffmpeg@0.12.10`, `@ffmpeg/core@0.12.6`) — requires internet on first export.

**Offline / self-hosted FFmpeg:** run `scripts/vendor-ffmpeg.sh` (needs internet once). It downloads `ffmpeg.js`, its worker chunk, `ffmpeg-core.js` and `ffmpeg-core.wasm` into `vendor/ffmpeg/`. Commit them if you want to host them yourself — but read the licensing note below first.

### How the exporter works

1. Preloads media and builds the audio mix-down (`OfflineAudioContext` → 48 kHz WAV).
2. Renders each frame to a canvas with the same renderer as the preview and writes JPEG (or PNG) frames into FFmpeg's in-memory file system.
3. Encodes in segments of up to 300 frames (scaled by resolution) with `libx264`, deleting frames after each segment to bound memory.
4. Concatenates segments with `-c copy` and muxes AAC audio, `+faststart`, BT.709 tagging.
5. Verifies the result by loading it into a `<video>` element.

## Browser support

Current Chrome, Edge, Firefox and Safari (Canvas 2D, IndexedDB, Web Audio, WebAssembly, Workers). Developed and automatically tested in Chromium. **Firefox and Safari have not been tested by the automated suite** — please report issues. M4A/AAC decoding depends on the browser/OS codecs.

## Limits & honest notes

* Encoding in WebAssembly is slow (single thread): expect minutes for long Full-HD projects. Keep the tab open and visible.
* Browser storage can be cleared by the user or by private-browsing mode; use **Projects → Back up this project** for a portable copy.
* The interface is English. Text inside videos supports right-to-left scripts (Arabic, Hebrew) with automatic direction detection; the app UI itself is not yet translated.
* SVG whiteboard drawing traces `path`, `line`, `polyline`, `polygon`, `rect`, `circle`, `ellipse`. `<use>`, text, masks, filters and gradients appear in the final colour fade-in (gradients are approximated by their first stop during tracing).
* There is no offline service worker; after the first load the app needs a connection only if FFmpeg is not vendored.
* Imported SVGs are sanitised (scripts, event handlers, external references and animation elements removed) and analysed inside a sandboxed iframe.

## Project layout

```
index.html            app shell (with Content-Security-Policy)
css/styles.css        theme and layout
js/core.js            config, utilities, event bus
js/model.js           project schema, normalisation, timing
js/anim.js            easing, keyframes, entrance/exit
js/renderer.js        deterministic frame renderer (+ transitions, hit testing)
js/infographics.js    chart/diagram engine, icons, themes
js/svgtools.js        SVG sanitising and vector analysis
js/media.js           import, validation, decoding, Sobel edges
js/audio.js           preview playback, mix-down, WAV
js/exporter.js        FFmpeg WASM export pipeline
js/store.js           IndexedDB persistence, portable files
js/state.js           state, undo/redo, actions, transport, autosave
js/ui-*.js            panels: stage, library, inspector, timeline, header, export
tests/                Playwright browser tests (see docs/TESTING.md)
docs/                 USER_GUIDE.md, ARCHITECTURE.md, TESTING.md
scripts/              vendor-ffmpeg.sh
```

## Tests

`npm install && npx playwright install chromium && npm test` — see [docs/TESTING.md](docs/TESTING.md).

## Licensing

Application code: © 2026 Nouvura, all rights reserved (see `LICENSE`; replace with the licence you choose). Third-party components and the **GPL** status of the FFmpeg core are described in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
