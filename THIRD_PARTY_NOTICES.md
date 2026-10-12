# Third-party notices

Nouvura Motion itself contains **no bundled third-party code**. The following components are loaded at runtime for MP4 export only.

| Component | Version | Licence | Notes |
| --- | --- | --- | --- |
| `@ffmpeg/ffmpeg` (ffmpeg.wasm wrapper) | 0.12.10 | MIT | JavaScript wrapper and worker |
| `@ffmpeg/core` (FFmpeg compiled to WebAssembly) | 0.12.6 | GPL-2.0-or-later (contains FFmpeg and libx264) | Single-thread core |
| FFmpeg | — | LGPL-2.1+/GPL-2.0+ depending on build | https://ffmpeg.org/legal.html |
| x264 | — | GPL-2.0-or-later | https://www.videolan.org/developers/x264.html |

## Important — please read before redistributing

* **Loading from the CDN** (default): your site does not host the GPL binaries; the visitor's browser downloads them from jsDelivr.
* **Vendoring into your repository** (`scripts/vendor-ffmpeg.sh`): you then *distribute* GPL-licensed binaries. You must comply with the GPL (provide the corresponding source or a written offer, keep licence texts). Source and licence details: https://github.com/ffmpegwasm/ffmpeg.wasm and https://github.com/ffmpegwasm/ffmpeg.wasm-core.
* **H.264 / AAC** are patent-encumbered formats. Depending on your jurisdiction and distribution model, patent licences (e.g. MPEG LA / Via LA) may apply to products that encode them. This is not legal advice — consult a lawyer for commercial distribution.
* Versions above were chosen at build time; verify licences against the versions you actually deploy.

No fonts, icons or images from third parties are bundled: UI icons and the 35 infographic icons are original simple vector paths, text uses fonts installed on the viewer's device.
