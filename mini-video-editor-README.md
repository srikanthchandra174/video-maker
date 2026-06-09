# Mini Video Editor 🎬

A browser-based video editor that turns **photos and short clips** into a shareable video — with transitions, captions, filters, effects, and music. **100% client-side**: no server, no upload, no dependencies, no build step. A single `index.html` deployed on GitHub Pages.

**▶ Live demo:** https://srikanthchandra174.github.io/video-maker/

---

## Overview

Add photos and short video clips, arrange them on a timeline, style them with filters and effects, lay a music track underneath, and export a video — all inside the browser. Because every operation runs on the client, **the user's media never leaves their device**, which makes it a privacy-respecting choice for personal photos and family videos.

It is intentionally a *focused* editor, not a clone of CapCut or Premiere — see [Limitations](#limitations) for the honest boundaries of what a browser can do.

## Features

**Timeline & media**
- Mix **photos and short video clips** in a single timeline
- Reorder items, remove items, add an optional **caption** to each
- **Ken Burns** zoom/pan on photos
- Per-clip **mute** toggle

**Look & feel**
- **Aspect ratios:** 9:16 (Reels/Stories), 4:5, 1:1, 16:9
- **Fit modes:** Fill (crop) or Fit (whole image over a blurred background)
- **Transitions:** Crossfade, Slide, Zoom, None — with adjustable length
- **11 filters:** None, B&W, Sepia, Vivid, Cool, Warm, Vintage, Noir, Dreamy, Invert, Cinematic
- **Manual adjustments:** brightness, contrast, saturation (stack on top of any filter)
- **Effects:** vignette, fade in/out from black, cinematic letterbox bars
- **Caption position:** top / center / bottom

**Audio**
- Upload a background music track (decoded to an audio buffer for reliable capture)
- Volume control and fade-out at the end

**Reliability & UX**
- **Autosave + resume** — the full project (media, captions, order, settings) is persisted to **IndexedDB** and restored on reload
- **HEIC detection** — iPhone `.heic`/`.heif` photos are caught with a clear message instead of failing silently
- **Image downscaling on import** + **object-URL revocation** to keep memory usage in check on mobile
- Live canvas **preview** before export

## Tech Stack

| Area | Technology |
|------|-----------|
| Language | Vanilla JavaScript (no framework) |
| Rendering | HTML5 **Canvas 2D** |
| Audio | **Web Audio API** (AudioBuffer, GainNode, MediaStreamDestination) |
| Video export | **MediaRecorder** + `canvas.captureStream()` |
| Persistence | **IndexedDB** |
| Styling | Plain CSS |
| Build / deploy | None — single static file on **GitHub Pages** |

No dependencies, no bundler, no transpiler. The entire app is one `index.html`.

## How It Works

1. **Import** — images are downscaled to ~2048px and stored as blobs; clips are loaded as `<video>` elements. Object URLs are revoked on removal to avoid leaks.
2. **Timeline** — each item gets a start time and duration; a render loop driven by a clock computes the active item and progress.
3. **Render** — each frame is drawn to a canvas: cover/contain fit, Ken Burns transform, CSS filter string (preset + adjustments), then a transition blend against a snapshot of the previous frame, then caption and overlay effects (vignette / bars / fade).
4. **Audio** — background music is decoded to an `AudioBuffer` and played through a `GainNode` into a `MediaStreamDestination`; clip audio is routed via `MediaElementSource`.
5. **Export** — the canvas video stream and the audio stream are combined into one `MediaStream` and recorded in real time with `MediaRecorder`.
6. **Persistence** — any change triggers a debounced save of the project (blobs + settings) to IndexedDB.

## Engineering Highlights

- **Privacy by architecture** — fully client-side; user media is never transmitted or stored on a server.
- **Memory management** — import-time downscaling and disciplined `URL.revokeObjectURL` cleanup to prevent out-of-memory crashes on mobile.
- **Resilient persistence** — IndexedDB autosave/restore of binary media plus project state, with debounced writes.
- **Defensive input handling** — explicit HEIC/unsupported-format detection with user-facing guidance rather than silent failure.
- **Reliable audio capture** — background audio is decoded to a buffer and fed into the recording graph, which captures more reliably than routing a media element directly.

## Getting Started

It's a single static file — no install required.

```bash
# Clone
git clone https://github.com/srikanthchandra174/video-maker.git
cd video-maker

# Run locally over http (NOT file:// — canvas/audio capture needs a real origin)
python -m http.server 8000
# open http://localhost:8000
```

**Deploy on GitHub Pages:** put the file in the repo root as `index.html` → Settings → Pages → Deploy from branch → `main` / root.

## Usage

1. Add photos and/or short clips.
2. Pick an aspect ratio, transition, filter, and effects; add captions.
3. (Optional) Upload a music track.
4. **Preview**, then **Export video**.
5. Play the result in Chrome/VLC, or convert to MP4 for Instagram.

## Limitations

These are honest constraints of building an editor in the browser — not bugs:

- **Real-time export.** Recording takes as long as the video plays, and the tab must stay in front. (`MediaRecorder` captures a live stream.)
- **Output format varies by browser.** WebM on Chromium/Firefox, MP4 on Safari/iOS. Instagram prefers MP4, so a WebM may need conversion.
- **Video-clip audio can be unreliable** in some browsers when mixed into the recording. A per-clip mute toggle is provided as a fallback; background music is the reliable audio path.
- **No pro features.** No multi-track timeline, keyframing, green-screen/chroma key, masking, speed ramping, or frame-accurate trimming — these require a native editor.
- **IndexedDB limits.** Large clips can exceed the browser's storage quota (surfaced as a save-failure message), and private/incognito modes may not persist data.

For advanced editing, a dedicated tool such as CapCut or DaVinci Resolve is the right choice; this project targets quick, private, in-browser edits.

## Roadmap

- **WebCodecs + MP4 muxer** for true MP4 output and frame-accurate **offline** rendering (decoupled from playback speed).
- **PWA** support (installable, offline-capable).
- Per-clip **trim** (in/out points).
- Split the single file into modular `index.html` / `app.js` / `style.css`.

## License

Released under the [MIT License](LICENSE).
