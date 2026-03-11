# DJZ-Onion-Layers

<!-- TINS Specification v1.0 -->
<!-- ZS:COMPLEXITY:MEDIUM -->
<!-- ZS:PRIORITY:HIGH -->
<!-- ZS:PLATFORM:WEB -->
<!-- ZS:LANGUAGE:JAVASCRIPT -->

---

## Description

DJZ-Onion-Layers is a fully client-side HTML5 web application for composing and aligning PNG frame sequences for hand-drawn or digitally-painted animation. It allows artists to load a batch of image frames, position and scale each one inside a uniform 1:1 square canvas container, and use "onion skin" overlay mode to compare a current frame against the previous frame at 50% opacity — the classic technique borrowed from traditional cel animation.

The target audience is animators, illustrators, and motion artists who work with frame-by-frame PNG sequences and need a lightweight, browser-based tool to clean up frame alignment before export. The app also supports batch background removal powered by MODNet, enabling transparent RGBA PNG output per frame. All processing occurs entirely in the browser; no files are ever uploaded to a server.

When finished, the user exports all adjusted frames as RGBA PNG files packed into a single ZIP archive, with a custom filename prefix and sequential frame numbering (e.g. `walk_001.png`, `walk_002.png`, …).

---

## Functionality

### Core Features

1. **Batch image load** — drag-and-drop zone or file picker accepting multiple PNG, JPG, and WebP files. Files are sorted by filename on load to establish frame order. Up to 200 frames supported.
2. **Uniform 1:1 square canvas container** — every frame is composited into a square canvas whose pixel size the user sets (e.g. 512×512, 1024×1024). The original image aspect ratio is always preserved; images are never stretched.
3. **Per-frame transform controls** — each frame can be independently positioned (X/Y offset) and scaled (uniform scale %) within the container. Transforms are applied non-destructively; the original pixel data is unchanged.
4. **Onion skin toggle** — a toggle button switches the current frame's canvas to show the current frame composited on top of the previous frame at 50% opacity. Aids alignment without switching views.
5. **Batch background removal with MODNet** — a single button runs MODNet inference (loaded via ONNX Runtime Web) on every frame, replacing the background with full transparency so the output is RGBA PNG.
6. **Frame timeline strip** — a horizontal scrollable strip of thumbnails at the bottom of the UI shows all frames in order. Clicking a thumbnail selects it as the current frame.
7. **Frame reordering** — frames can be dragged to a new position in the timeline strip to re-sequence them before export.
8. **Custom export prefix and ZIP download** — user types a filename prefix (e.g. `walk`). On export, all frames are saved as `{prefix}_{001..NNN}.png` inside a single ZIP file.
9. **Container size control** — numeric input (pixels) sets the width and height of the square export canvas (minimum 64px, maximum 4096px, default 512px).
10. **Per-frame status badge** — each thumbnail in the timeline strip shows a status: `Loaded`, `BG Removed`, or `Error`.

---

### UI Layout (ASCII)

```
+=====================================================================+
|  🎞 DJZ-Onion-Layers                                    [?] Help  |
+=====================================================================+
|  TOOLBAR                                                            |
|  [📂 Load Frames]  Container: [512] px   [🗑 Remove BG (MODNet)]  |
|  Export prefix: [_________]   [⬇ Export ZIP]                       |
+---------------------------------------------------------------------+
|  CANVAS AREA                               TRANSFORM PANEL          |
|  ┌──────────────────────────────┐          Frame: 003 / 024         |
|  │                              │          ┌──────────────────────┐ |
|  │   ┌──────────────────────┐   │          │ X Offset:  [  0 ] px │ |
|  │   │  [current frame]     │   │          │ Y Offset:  [  0 ] px │ |
|  │   │  [prev frame @ 50%]  │   │          │ Scale:    [100  ] %  │ |
|  │   │  (onion skin)        │   │          │ [Reset Transform]    │ |
|  │   └──────────────────────┘   │          └──────────────────────┘ |
|  │                              │          [👁 Onion Skin: ON/OFF]  |
|  └──────────────────────────────┘                                   |
+---------------------------------------------------------------------+
|  TIMELINE STRIP (horizontal scroll)                                 |
|  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐            |
|  │ 001  │ │ 002  │▶│ 003  │ │ 004  │ │ 005  │ │ 006  │  …         |
|  │[img] │ │[img] │ │[img] │ │[img] │ │[img] │ │[img] │            |
|  │Loaded│ │BG Rmv│ │Loaded│ │Loaded│ │Error │ │Loaded│            |
|  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘ └──────┘            |
+=====================================================================+
```

**Legend:** ▶ = currently selected frame. Timeline thumbnails are 80×80px. Active frame has a blue border. Onion Skin toggle button lives in the Transform Panel.

---

### User Flows

#### Flow A — Basic Frame Alignment

1. User opens app in a browser.
2. User clicks **Load Frames** or drags a folder's worth of PNG files onto the drop zone.
3. Files are sorted by filename and appear in the timeline strip as thumbnails, all with status `Loaded`.
4. User sets container size to `1024` px.
5. User clicks frame `001` in the timeline. It appears centred in the canvas.
6. User clicks frame `002`. It appears in the canvas. User enables **Onion Skin**; frame `001` appears behind it at 50% opacity.
7. User adjusts **X Offset**, **Y Offset**, and **Scale** in the Transform Panel until the two frames align correctly.
8. User steps through all frames, adjusting alignment as needed.
9. User types `walk` in the export prefix field and clicks **Export ZIP**.
10. A file `walk.zip` downloads containing `walk_001.png`, `walk_002.png`, … all as RGBA PNG at 1024×1024.

#### Flow B — Background Removal then Align

1. User loads frames as in Flow A.
2. User clicks **Remove BG (MODNet)**. A progress bar appears; each frame is processed sequentially.
3. Each thumbnail status updates to `BG Removed` when done. Thumbnails update to show transparent backgrounds on a checker pattern.
4. User then aligns frames as in Flow A steps 5–9.
5. Exported ZIPs contain RGBA PNGs with backgrounds removed and alignment applied.

#### Flow C — Reorder Frames

1. User loads frames and notices two frames are out of order.
2. User drags a thumbnail in the timeline strip to a new position.
3. Frame numbers update to reflect the new sequence order.
4. User exports; the ZIP reflects the reordered sequence.

#### Flow D — Error on Load

1. A file is corrupt or not a supported image format.
2. Its thumbnail shows status `Error: Could not load image`.
3. A tooltip on hover shows the filename and error reason.
4. User can click a small **✕** on the thumbnail to remove that frame from the sequence.

---

### Edge Cases

- **Non-square source images** — images are fitted into the square container using `object-fit: contain` logic: the largest dimension fills the container, centremost by default. Scale and offset can adjust from there. Images are never distorted.
- **Single frame loaded** — Onion Skin mode is automatically disabled and the toggle is greyed out with tooltip `"Need at least 2 frames for Onion Skin"`.
- **Frame 001 selected with Onion Skin ON** — no previous frame exists; the canvas shows only the current frame with a subtle `"No previous frame"` label in the corner. No error is thrown.
- **MODNet model not yet loaded** — **Remove BG** button is disabled with tooltip `"Loading MODNet model…"` until the ONNX model resolves.
- **Zero frames loaded** — **Remove BG**, **Export ZIP**, and **Onion Skin** are all disabled.
- **Container size out of range** — values below 64 are clamped to 64; values above 4096 are clamped to 4096. An inline warning is shown.
- **Export prefix empty** — defaults to `frame` producing `frame_001.png`, `frame_002.png`, etc.
- **Export prefix with invalid filename characters** — characters `/\:*?"<>|` are stripped silently before use.
- **Very large frames (e.g. 4K source + 4096 container)** — processing is done one frame at a time to avoid browser memory exhaustion. A progress indicator is shown.
- **Browser without WebGL/WebAssembly** — ONNX Runtime Web requires WASM; if unavailable, show a banner: `"Background removal requires WebAssembly support. Please use a modern browser."` and disable the Remove BG button.

---

## Technical Implementation

### Architecture

Pure client-side single-page application. No backend. No server uploads. Three logical layers:

```
┌────────────────────────────────────────────────────────┐
│  UI Layer (HTML5 + CSS3 + Vanilla JS)                  │
│  Drop zone, canvas display, timeline strip, controls   │
├────────────────────────────────────────────────────────┤
│  Background Removal Layer (ONNX Runtime Web + MODNet)  │
│  Model loading, inference, alpha mask compositing      │
├────────────────────────────────────────────────────────┤
│  Canvas / Export Layer (Canvas API + JSZip)            │
│  Frame compositing, transform application, PNG export  │
└────────────────────────────────────────────────────────┘
```

**Files produced by an LLM implementing this spec:**

```
index.html          ← single HTML file (all CSS and JS inline, or split as below)
style.css           ← application styles
app.js              ← main application logic, state, event wiring
timeline.js         ← timeline strip rendering and drag-reorder logic
canvas-engine.js    ← frame compositing, transform math, onion skin render
bg-remover.js       ← MODNet ONNX inference, alpha mask extraction
export.js           ← ZIP generation, canvas-to-PNG blob, filename logic
```

All files may be inlined into a single `index.html` for portability.

---

### Dependencies (CDN)

```html
<!-- ONNX Runtime Web (WebAssembly backend for MODNet) -->
<script src="https://cdn.jsdelivr.net/npm/onnxruntime-web@1.17.0/dist/ort.min.js"></script>

<!-- JSZip for batch ZIP download -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>

<!-- FileSaver.js for triggering download -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/FileSaver.js/2.0.5/FileSaver.min.js"></script>
```

MODNet ONNX model file (`modnet.onnx`) must be served from the same origin or a CORS-permitting CDN. The model is fetched once and cached in memory for the session. Recommended source: a self-hosted copy of the MODNet ONNX export (approximately 25 MB).

No build step required.

---

### Data Models

#### FrameItem (one per loaded image file)

```javascript
{
  id: string,                      // UUID generated on load
  file: File,                      // Original File object
  originalImage: HTMLImageElement, // Loaded DOM image element
  naturalWidth: number,            // Source image width in px
  naturalHeight: number,           // Source image height in px
  rgbaCanvas: HTMLCanvasElement | null, // Canvas after BG removal (null until processed)
  transform: {
    x: number,                     // X offset in px within container (default 0)
    y: number,                     // Y offset in px within container (default 0)
    scale: number,                 // Uniform scale factor 0.01–10.0 (default 1.0)
  },
  sequenceIndex: number,           // Current position in the timeline (0-based)
  status: 'loaded' | 'bg_removed' | 'error',
  errorMessage: string | null,
  thumbnailDataUrl: string,        // 80×80 data URL for timeline strip display
}
```

#### AppState (global application state)

```javascript
{
  frames: FrameItem[],             // Ordered array of all loaded frames
  selectedIndex: number,           // Index of currently viewed frame
  containerSize: number,           // Square canvas size in px (default 512)
  onionSkinEnabled: boolean,       // Whether previous-frame overlay is active
  exportPrefix: string,            // Filename prefix for ZIP export (default 'frame')
  modnetSession: ort.InferenceSession | null, // Loaded ONNX session
  modnetLoading: boolean,          // True while model is loading
  bgRemovalProgress: number,       // 0–100 during batch BG removal
  bgRemovalRunning: boolean,
}
```

---

### Key Algorithms

#### 1. Fitting a Frame into the Square Container

The container is always `containerSize × containerSize` pixels. When compositing a frame, the source image is scaled so its largest dimension equals `containerSize`, preserving aspect ratio (contain fit). The user's `transform.scale` is applied multiplicatively on top of this base fit scale. The user's `transform.x` and `transform.y` are pixel offsets applied after scaling.

```javascript
// canvas-engine.js
function computeDrawParams(frame, containerSize) {
  const { naturalWidth, naturalHeight, transform } = frame;

  // Base contain-fit scale: largest dimension fills the container
  const baseFit = Math.min(containerSize / naturalWidth, containerSize / naturalHeight);

  // Apply user scale on top
  const finalScale = baseFit * transform.scale;

  const drawW = naturalWidth * finalScale;
  const drawH = naturalHeight * finalScale;

  // Centre by default, then apply user offset
  const drawX = (containerSize - drawW) / 2 + transform.x;
  const drawY = (containerSize - drawH) / 2 + transform.y;

  return { drawX, drawY, drawW, drawH };
}
```

#### 2. Rendering the Main Canvas

The main display canvas is always `containerSize × containerSize`. On each render call:

```javascript
// canvas-engine.js
function renderCanvas(state, displayCanvas) {
  const { frames, selectedIndex, containerSize, onionSkinEnabled } = state;
  const ctx = displayCanvas.getContext('2d');

  displayCanvas.width = containerSize;
  displayCanvas.height = containerSize;

  // Checkerboard background (indicates transparency)
  drawCheckerboard(ctx, containerSize);

  const currentFrame = frames[selectedIndex];
  if (!currentFrame) return;

  // --- Onion skin: render previous frame at 50% opacity first ---
  if (onionSkinEnabled && selectedIndex > 0) {
    const prevFrame = frames[selectedIndex - 1];
    ctx.save();
    ctx.globalAlpha = 0.5;
    drawFrame(ctx, prevFrame, containerSize);
    ctx.restore();
  }

  // --- Render current frame at full opacity ---
  ctx.globalAlpha = 1.0;
  drawFrame(ctx, currentFrame, containerSize);

  // Label if onion skin active but no previous frame
  if (onionSkinEnabled && selectedIndex === 0) {
    ctx.fillStyle = 'rgba(255,255,255,0.4)';
    ctx.font = '12px system-ui';
    ctx.fillText('No previous frame', 8, containerSize - 8);
  }
}

function drawFrame(ctx, frame, containerSize) {
  // Prefer rgbaCanvas (post BG removal) over original image
  const source = frame.rgbaCanvas || frame.originalImage;
  const { drawX, drawY, drawW, drawH } = computeDrawParams(frame, containerSize);
  ctx.drawImage(source, drawX, drawY, drawW, drawH);
}
```

#### 3. MODNet Background Removal

MODNet accepts a 512×512 RGB float32 tensor and produces a 512×512 alpha matte. The matte is applied to the original image at its native resolution using a temporary canvas.

```javascript
// bg-remover.js
async function removeBackground(frame, ortSession) {
  const INPUT_SIZE = 512;

  // 1. Draw the source image onto a 512×512 canvas for inference input
  const inputCanvas = document.createElement('canvas');
  inputCanvas.width = INPUT_SIZE;
  inputCanvas.height = INPUT_SIZE;
  const inputCtx = inputCanvas.getContext('2d');
  inputCtx.drawImage(frame.originalImage, 0, 0, INPUT_SIZE, INPUT_SIZE);
  const imageData = inputCtx.getImageData(0, 0, INPUT_SIZE, INPUT_SIZE);

  // 2. Build Float32 tensor [1, 3, 512, 512], normalise to [-1, 1]
  const float32 = new Float32Array(3 * INPUT_SIZE * INPUT_SIZE);
  const mean = [0.5, 0.5, 0.5];
  const std  = [0.5, 0.5, 0.5];
  for (let i = 0; i < INPUT_SIZE * INPUT_SIZE; i++) {
    float32[i]                          = (imageData.data[i * 4]     / 255 - mean[0]) / std[0]; // R
    float32[i + INPUT_SIZE * INPUT_SIZE]     = (imageData.data[i * 4 + 1] / 255 - mean[1]) / std[1]; // G
    float32[i + 2 * INPUT_SIZE * INPUT_SIZE] = (imageData.data[i * 4 + 2] / 255 - mean[2]) / std[2]; // B
  }
  const tensor = new ort.Tensor('float32', float32, [1, 3, INPUT_SIZE, INPUT_SIZE]);

  // 3. Run inference
  const results = await ortSession.run({ input: tensor });
  const matteTensor = results[Object.keys(results)[0]]; // first output
  const matteData = matteTensor.data; // Float32Array, values 0.0–1.0

  // 4. Scale matte back to original image dimensions and apply as alpha
  const { naturalWidth: W, naturalHeight: H } = frame;
  const outputCanvas = document.createElement('canvas');
  outputCanvas.width = W;
  outputCanvas.height = H;
  const outCtx = outputCanvas.getContext('2d');
  outCtx.drawImage(frame.originalImage, 0, 0);
  const outImageData = outCtx.getImageData(0, 0, W, H);

  for (let y = 0; y < H; y++) {
    for (let x = 0; x < W; x++) {
      // Sample matte at scaled coordinates (bilinear would be ideal; nearest-neighbour is acceptable)
      const mx = Math.round((x / W) * (INPUT_SIZE - 1));
      const my = Math.round((y / H) * (INPUT_SIZE - 1));
      const alpha = matteData[my * INPUT_SIZE + mx]; // 0.0 to 1.0
      outImageData.data[(y * W + x) * 4 + 3] = Math.round(alpha * 255);
    }
  }
  outCtx.putImageData(outImageData, 0, 0);

  frame.rgbaCanvas = outputCanvas;
  frame.status = 'bg_removed';
}
```

#### 4. Batch Background Removal Queue

```javascript
// bg-remover.js
async function removeAllBackgrounds(state, onProgress) {
  const { frames, modnetSession } = state;
  state.bgRemovalRunning = true;

  for (let i = 0; i < frames.length; i++) {
    const frame = frames[i];
    if (frame.status === 'error') continue;
    try {
      await removeBackground(frame, modnetSession);
    } catch (err) {
      frame.status = 'error';
      frame.errorMessage = err.message;
    }
    onProgress(Math.round(((i + 1) / frames.length) * 100));
    updateThumbnail(frame); // re-render thumbnail with transparent bg
    // Yield to UI thread
    await new Promise(r => setTimeout(r, 0));
  }

  state.bgRemovalRunning = false;
}
```

#### 5. Exporting Frames to ZIP

Each frame is composited into a new `containerSize × containerSize` offscreen canvas applying its transforms, then exported as PNG.

```javascript
// export.js
async function exportZip(state) {
  const { frames, containerSize, exportPrefix } = state;
  const zip = new JSZip();
  const prefix = sanitisePrefix(exportPrefix) || 'frame';

  for (let i = 0; i < frames.length; i++) {
    const frame = frames[i];
    if (frame.status === 'error') continue;

    // Offscreen canvas at export resolution
    const offscreen = document.createElement('canvas');
    offscreen.width = containerSize;
    offscreen.height = containerSize;
    const ctx = offscreen.getContext('2d');

    // Transparent background
    ctx.clearRect(0, 0, containerSize, containerSize);

    // Draw the frame (uses rgbaCanvas if available, otherwise originalImage)
    drawFrame(ctx, frame, containerSize);

    // Convert to PNG blob (preserves alpha)
    const blob = await new Promise(resolve =>
      offscreen.toBlob(resolve, 'image/png')
    );

    const frameNumber = String(i + 1).padStart(3, '0');
    zip.file(`${prefix}_${frameNumber}.png`, blob);
  }

  const content = await zip.generateAsync({ type: 'blob' });
  saveAs(content, `${prefix}.zip`);
}

function sanitisePrefix(raw) {
  return raw.replace(/[/\\:*?"<>|]/g, '').trim();
}
```

#### 6. Timeline Strip Drag-to-Reorder

```javascript
// timeline.js
// Each thumbnail <div> has draggable="true" and carries data-index attribute.
// dragstart: store the dragged index in event.dataTransfer.
// dragover: add a visual insertion indicator (blue left border) on the hovered thumbnail.
// drop: read dragged index, compute drop target index, splice frames array to reorder.
// After reorder: re-assign sequenceIndex values for all frames, re-render strip.
// selectedIndex is updated to follow the moved frame.
```

#### 7. MODNet ONNX Model Loading

```javascript
// bg-remover.js
async function loadModnet(modelUrl) {
  // modelUrl: path to modnet.onnx, e.g. './models/modnet.onnx'
  const session = await ort.InferenceSession.create(modelUrl, {
    executionProviders: ['wasm'], // WebAssembly backend
  });
  return session;
}
```

The model is loaded once on app startup (non-blocking). The **Remove BG** button is disabled until the session is ready.

---

### Checkerboard Background (Transparency Indicator)

The main canvas background always shows a standard checkerboard pattern to visually indicate transparency. This is drawn using a small tile pattern on a background canvas.

```javascript
// canvas-engine.js
function drawCheckerboard(ctx, size, tileSize = 16) {
  const light = '#cccccc';
  const dark  = '#999999';
  for (let y = 0; y < size; y += tileSize) {
    for (let x = 0; x < size; x += tileSize) {
      ctx.fillStyle = ((x / tileSize + y / tileSize) % 2 === 0) ? light : dark;
      ctx.fillRect(x, y, tileSize, tileSize);
    }
  }
}
```

---

### Style Guide

| Token           | Value                        |
|-----------------|------------------------------|
| Background      | `#0F172A` (dark navy)        |
| Surface         | `#1E293B`                    |
| Card bg         | `#263548`                    |
| Accent          | `#63B3ED` (blue)             |
| Onion skin tint | `rgba(255, 160, 80, 0.5)` — warm amber at 50%, overlaid on canvas |
| Success         | `#48BB78` (green)            |
| Warning         | `#F6AD55` (amber)            |
| Error           | `#FC8181` (red)              |
| Text primary    | `#F1F5F9`                    |
| Text muted      | `#94A3B8`                    |
| Font            | `system-ui, sans-serif`      |
| Border radius   | `8px` (panels), `4px` (buttons) |
| Timeline thumb  | `80×80px`, `4px` border on active (`#63B3ED`) |
| Drop zone border| `2px dashed #63B3ED`, animated dash on hover |

Active frame thumbnail has a solid blue border (`#63B3ED`, 3px). BG-removed thumbnails show a checker pattern behind the subject.

Buttons use accent background with white text. Disabled state: `opacity 0.4`, `cursor: not-allowed`.

---

## Accessibility Requirements

- All interactive controls are keyboard focusable (Tab order follows visual layout order).
- Drop zone has `role="button"` and `aria-label="Frame drop zone. Press Enter or Space to open file picker."`.
- Timeline strip thumbnails have `aria-label="Frame {N}: {filename}, status: {status}"` and are focusable with arrow-key navigation between them.
- Onion Skin toggle is an ARIA toggle button: `aria-pressed="true|false"`.
- Progress during BG removal is announced via an `aria-live="polite"` region.
- Canvas elements have `aria-label="Animation preview canvas"` on their wrapper.
- Colour is never the sole indicator of status — all badges include text labels.
- Escape key closes any open modals or tooltips.

---

## Performance Goals

- Model load time: < 8 seconds on a 10 Mbps connection (MODNet ONNX is ~25 MB).
- BG removal per frame: < 2 seconds per frame on a mid-range laptop (WASM backend).
- Canvas render (compositing + onion skin): < 30 ms per frame at 1024×1024.
- Timeline strip with 200 thumbnails: renders within 500 ms using deferred thumbnail generation.
- Export ZIP of 24 frames at 1024×1024 RGBA PNG: completes within 15 seconds.
- Memory: ONNX tensors are not persisted beyond each inference call. All per-frame canvases are reused rather than recreated.

---

## Testing Scenarios

| Scenario | Expected Result |
|---|---|
| Load 24 PNGs sorted by filename | Timeline shows 24 thumbnails in correct order, status = Loaded |
| Select frame 5, enable Onion Skin | Frame 4 shown at 50% opacity beneath frame 5 |
| Select frame 1, enable Onion Skin | Frame shown alone, "No previous frame" label visible |
| Adjust X/Y offset on a frame | Canvas updates immediately; export reflects the offset |
| Adjust scale on a frame | Image scales within container, aspect ratio preserved |
| Reset Transform | X=0, Y=0, Scale=100% restored for current frame |
| Click Remove BG | Progress bar advances; thumbnails update to show transparent BG |
| Drag frame 3 thumbnail to position 1 | Sequence reorders; frame numbers update accordingly |
| Export with prefix "walk" | ZIP downloads as walk.zip containing walk_001.png … |
| Export prefix left empty | ZIP uses "frame" as prefix: frame_001.png … |
| Export prefix contains `*` | Character stripped; export proceeds without error |
| Load a corrupt image file | Thumbnail shows "Error" badge; other frames unaffected |
| Set container to 2048px and export | All PNGs are 2048×2048 with correct transforms |
| Set container to 50px (below min) | Clamped to 64px; warning shown |

---

## Extended Features (Optional Enhancements)

- **Frame duplication** — right-click a thumbnail to duplicate the frame (copy transform + source image).
- **Per-frame onion skin opacity** — a slider to adjust the onion skin opacity from 10%–90% instead of fixed 50%.
- **Multi-frame onion skin** — show up to 3 previous frames at decreasing opacity (e.g. 50%, 30%, 15%).
- **Fit / Fill / Free transform modes** — toggle between contain-fit (default), cover-fill, or free (unconstrained aspect ratio) per frame.
- **Global transform nudge** — apply a constant X/Y/scale offset to all frames simultaneously (useful for fixing a systematic misalignment).
- **Playback preview** — a play button cycles through frames at a user-specified FPS (1–30) in the canvas area for a real-time animation preview.
- **Frame note labels** — small editable text labels per thumbnail for annotating key frames.
- **Undo/Redo stack** — Ctrl+Z / Ctrl+Y for transform changes.
- **Import ZIP** — load a ZIP of PNGs directly as a frame sequence without extracting first.
