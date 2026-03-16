# BounceX Editor

A standalone path editor for [BounceX-Viewer](https://github.com/Alunacoz/BounceX-Viewer). Sync video playback with marker placement and export `.bx` path files. This tool and summary (right now) have been written with generative AI so it may not be perfectly accurate. For most questions that are not answered here, please contact me (Alunacoz) on the [DH Discord Server](https://discord.gg/u6CZ3Zm4PC) in #bouncex-player or DMs!

---

## Getting Started

1. Run `./StartEditor.sh` or `./StartEditor.bat` and open the link in any browser.
2. Click **Load Video** (or drag a video file onto the window)
3. Click **Open path…** to load an existing `.bx` file (optional — supports both plain `.bx` and versioned `.bx` with effects)
4. Use the timeline and controls to place and edit markers
5. Click **Export .bx** to save

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `Space` | Play / Pause |
| `M` | Add marker at current frame |
| `Delete` / `Backspace` | Delete selected markers or selected effect |
| `←` | Step 1 frame back, **or** set all selected markers to depth 0.0 |
| `→` | Step 1 frame forward, **or** set all selected markers to depth 1.0 |
| `↑` | Set all selected markers to depth 0.5 |
| `Shift+←` / `Shift+→` | Step 10 frames |
| `[` | Jump to previous marker |
| `]` | Jump to next marker |
| `R` | Toggle record mode (starts video playback if paused) |
| `Ctrl+A` | Select all markers |
| `Ctrl+Z` | Undo |
| `Ctrl+Y` / `Ctrl+Shift+Z` | Redo |
| `Ctrl+C` | Copy selected markers or selected effect |
| `Ctrl+X` | Cut selected markers or selected effect |
| `Ctrl+V` | Paste at playhead position |
| `Escape` | Exit record mode / clear selection |

Arrow keys set depth when **one or more** markers are selected (not just one). When nothing is selected they step the playhead.

---

## Timeline Controls

### Marker Timeline

- **Click** empty space → seek and clear selection
- **Click** a marker → select it; seek on release (the playhead does **not** move while dragging, so you can pre-position it as a snap target)
- **Drag** a marker → move it; snaps to playhead within 10px
- **Shift+Click** → range-select from last click
- **Ctrl+Click** → toggle individual markers
- **Scroll wheel** → scroll; **Ctrl+Scroll** → zoom
- Drag the **resize handle** above the toolbar to change height

Marker diamonds sit at their depth position on the waveform (depth 0 = top, depth 1 = bottom), white fill with accent/teal outline when selected or nearest.

### Effects Timeline

- **Double-click** empty row → open effect type picker
- **Drag** a block horizontally → move it; drag vertically to change layer
- **Drag** left/right edge → resize duration
- **Click** → select and open properties
- **Del** → delete selected effect

Effects on the **same layer** collide — they cannot overlap. Dragging one into another pushes up to the neighbour's edge. Drag far enough past a neighbour and the block passes through to the other side; the barrier then applies from that side when you drag back. To overlap effects intentionally, place them on **different layers**.

---

## Record Mode

Press `R` to enter record mode. If the video is paused it starts automatically. While playing:

| Key | Depth stamped |
|-----|---------------|
| `←` | 0.0 (no stroke) |
| `↑` | 0.5 (half depth) |
| `→` | 1.0 (full depth) |

Record mode exits automatically when the video pauses or ends. Press `R` or `Escape` to exit manually.

---

## Generate Cycle

The **Generate Cycle** button (clock icon, in the Markers panel header) places depth-0 markers at a given BPM starting from the playhead.

| Field | Description |
|-------|-------------|
| BPM | Beats per minute. Supports decimals (e.g. 122.4). Use **½** / **×2** to halve or double. |
| Offset beats | Skip this many beats before placing the first marker (useful for pickup bars). |
| Count | Number of markers to place. Leave blank to fill the entire timeline from the playhead. |

**Fractional BPM accuracy:** each beat's ideal position is `startFrame + n × framesPerBeat` computed as a float and rounded to the nearest integer. This means rounding error never compounds — each beat independently corrects for the previous one. At 122 BPM the maximum drift is well under half a frame across hundreds of beats.

---

## Effects

The Effects timeline is hidden by default. Click **▸ EFFECTS** in the timeline toolbar to reveal it. Effects are stored inside the exported `.bx` file and rendered by BounceX-Viewer at playback time.

Last-used settings per effect type are remembered and pre-filled when you create the next effect of the same type.

### Text Overlay

Displays fading text on the path canvas. Size and position are percentages of the **path area**, so they look consistent across all canvas sizes and viewer modes.

| Property | Description |
|----------|-------------|
| Text | Content; `\n` for line breaks. Long text auto-shrinks to fit. |
| Font | Built-in list or upload a custom `.ttf`/`.otf`/`.woff` |
| Size | % of path area height (default 50%) |
| Color | Text colour |
| Opacity | 0–1, multiplied by fade alpha |
| Position X / Y | % of path area (0,0 = top-left) |
| Fade In / Fade Out | Duration in frames |

### Path Color

Smoothly transitions the waveform and ball colour using linear RGB interpolation.

| Property | Description |
|----------|-------------|
| Path | Target waveform colour |
| Ball | Target ball colour |
| Fade In / Fade Out | Blend duration in frames |

### Path Speed

Changes how fast the waveform scrolls. Transition is smooth — speed is lerped using the fade alpha, and each frame's x-position is integrated individually so only the affected zone stretches.

| Property | Description |
|----------|-------------|
| Speed | 0.5×, 0.75×, 1×, 1.25×, 1.5×, 1.75×, 2×, 2.5×, 3×, 3.5×, 4× |
| Fade In / Fade Out | Frames to ramp up / down |

---

## Export

Click **Export .bx** to save. In Chrome/Edge a native OS save dialog appears with a pre-filled name you can edit. In Firefox a prompt asks for a filename before downloading.

The internal file structure depends on whether effects are present:

| Condition | Structure |
|-----------|-----------|
| No effects | Plain `{"frame": [depth, trans, ease, aux], ...}` — identical to original `.bx` |
| Has effects | `{"version": 2, "markers": {...}, "effects": [...]}` |

**Open path…** accepts both automatically. `.bx2` files (older extension) are also accepted.

---

## Copy / Paste

`Ctrl+C` copies selected markers or the selected effect. `Ctrl+V` pastes at the playhead:

- **Markers** — offset so the first marker lands at the playhead; occupied frames are skipped
- **Effects** — start at the playhead, same duration, auto-assigned to a free layer

`Ctrl+X` copies and deletes. A flash message in the toolbar confirms each operation.

---

## Undo / Redo

All mutations are undoable (add/delete/move markers, depth/transition/ease changes, add/delete effects). Stack holds 100 snapshots. `Ctrl+Z` / `Ctrl+Y` or the toolbar buttons.

---

## Layout Persistence

Panel widths, timeline heights, and effects panel visibility are saved to `localStorage` and restored on next open.

To change the default sizes for a fresh session, edit the constants near the top of `editor.js`:

```js
const TL_H_DEFAULT    = 220   // marker timeline height (px)
const PROPS_W_DEFAULT = 272   // right panel width (px)
```

---

## Running Locally

### Linux / macOS

```bash
chmod +x StartEditor.sh
./StartEditor.sh
```

### Windows

Double-click `StartEditor.bat`. Python 3 is required; the script offers to install it via `winget` if not found.

Both scripts read the port from `config.json`:

```json
{ "httpPort": 8003 }
```

Then open `http://localhost:8003` (or `http://localhost:8003/editor.html` if not renamed to `index.html`).

---

## File Format Reference

### Plain `.bx`

```json
{
  "0":   [0.0, 1, 2, 0],
  "120": [1.0, 1, 2, 0],
  "360": [0.0, 4, 1, 0]
}
```

Keys are frame numbers (strings). Values are `[depth, trans, ease, aux]`.

| Index | Field | Type | Description |
|-------|-------|------|-------------|
| 0 | `depth` | float 0–1 | Stroke depth |
| 1 | `trans` | int 0–11 | Godot 4 TransitionType (see table below) |
| 2 | `ease` | int 0–3 | Godot 4 EaseType (see table below) |
| 3 | `aux` | float | Reserved, always `0` |

**Transition types:** 0=Linear, 1=Sine, 2=Quint, 3=Quart, 4=Quad, 5=Expo, 6=Elastic, 7=Cubic, 8=Circ, 9=Bounce, 10=Back, 11=Spring

**Ease types:** 0=In, 1=Out, 2=In-Out, 3=Out-In

**Interpolation** between markers A→B: `depth = A.depth + (B.depth - A.depth) * godotEase(t, B.trans, B.ease)` where `t = (f - fA) / (fB - fA)`. Parameters come from the **destination** marker B.

---

### Versioned `.bx` (with effects)

```json
{
  "version": 2,
  "markers": { "0": [0.0, 1, 2, 0], "120": [1.0, 1, 2, 0] },
  "effects": [
    {
      "id": "e1", "type": "text", "layer": 0,
      "startFrame": 60, "endFrame": 300, "fadeIn": 30, "fadeOut": 30,
      "text": "Hello", "font": "Rajdhani", "fontSize": 50,
      "color": "#ffffff", "opacity": 1.0, "posX": 50, "posY": 50
    },
    {
      "id": "e2", "type": "pathColor", "layer": 0,
      "startFrame": 120, "endFrame": 240, "fadeIn": 60, "fadeOut": 60,
      "pathColor": "#e05050", "ballColor": "#1a5fb4"
    },
    {
      "id": "e3", "type": "pathSpeed", "layer": 0,
      "startFrame": 300, "endFrame": 600, "fadeIn": 60, "fadeOut": 60,
      "speed": 2.0
    }
  ]
}
```

**Parsing (one-liner for developers who only need markers):**

```js
const parsed  = JSON.parse(fileContents)
const markers = parsed.version === 2 ? parsed.markers : parsed
// `markers` is now a plain .bx-compatible object — ignore `effects` entirely
```

#### Common effect fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique within the file |
| `type` | string | `"text"`, `"pathColor"`, or `"pathSpeed"` |
| `layer` | int | Track layer; effects on the same layer must not overlap |
| `startFrame` | int | First active frame (inclusive) |
| `endFrame` | int | Last active frame (inclusive) |
| `fadeIn` | int | Frames to ramp 0 → full |
| `fadeOut` | int | Frames to ramp full → 0 |

**Fade alpha:** `alpha = clamp(min(elapsed/fadeIn, (duration-elapsed)/fadeOut), 0, 1)` — supports fractional frames for smooth sub-frame interpolation.

#### `type: "text"` extra fields

| Field | Description |
|-------|-------------|
| `text` | Display string; `\n` = line break. Rendered text auto-shrinks to fit canvas width. |
| `font` | CSS font-family |
| `fontSize` | % of path area height |
| `color` | CSS hex colour |
| `opacity` | 0–1, multiplied by fade alpha |
| `posX` / `posY` | % of path area width / height |

#### `type: "pathColor"` extra fields

`pathColor` and `ballColor` — CSS hex. Blended as `lerp(default, target, alpha)` in linear RGB.

#### `type: "pathSpeed"` extra fields

`speed` — multiplier 0.5–4.0. Effective speed = `lerp(1.0, speed, alpha)`. Waveform x-positions integrate per-frame speed from the playhead so only the affected zone stretches.

---

## Browser Requirements

- Chrome 86+ or Edge 86+ — native save dialog, best experience
- Firefox — fallback filename prompt + download link
- Open via local web server or `file://`
- No backend, no install — three files: `editor.html`, `editor.css`, `editor.js`
