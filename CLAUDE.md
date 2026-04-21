# CLAUDE.md — Phantomra.in Project Memory

## What this project is
A single-file portfolio site for **Phantomra.in** — no build step, no frameworks, no external JS.
Everything lives in `index.html`. Google Fonts (Space Grotesk + Space Mono) load from CDN.
The file is fully self-contained. Serve locally with `npx serve .` (required for audio — `file://` blocks AudioContext).

---

## Aesthetic intent
Dark brutalist / generative art. The overall feel: **"a terminal that became a living landscape."**
- Black background, white typography
- Monochrome with **sparse red accents** — only on CTA hover states and card hover borders
- Monospace/grotesque font pairing throughout
- Everything feels deliberate, minimal, slightly cold

---

## Layout & sections

### Chrome frame
- 1px white border inset ~12px from viewport edges (like a monitor bezel)
- **Top bar**: hamburger icon left · "PHANTOMRA.IN" centre · dash/minimise right — decorative only
- **Bottom status bar**: `©PHANTOMRA.IN` left · `ZPHANTOMRA.IN 2026` centre · player + live clock right (format: `MM/DD::HHMM+SS`)
- All chrome text: Space Mono, 10px, opacity 0.6

### Hero (`#hero`)
- Large bold grotesque headline: **"Phantomra.in"** — types in character by character on load
  - Typewriter speed: 145ms per character, 420ms initial delay
  - A block cursor (`#hero-cursor`) shows solid during typing, switches to blinking once done
  - Cursor sized with `clamp(18px, 0.55em, 66px)` — thick terminal-style block
  - `white-space: nowrap` on `.hero-title` so glitch chars never cause line wrap
- Subtitle (Space Mono, monospace caps): `// The drips before the dream`
- CTA: `ENTER THE MESH` — white border, transparent bg; hover inverts to white bg / black text + red border glow

### Work (`#work`)
- Section header: `// WORK`
- 3 project cards in responsive grid (auto-fit, min 280px)
- Each card: thin white border, project number (PRJ_001 etc.), title, descriptor, `VIEW →` link
- Hover: border turns red, faint red background tint

### Contact (`#contact`)
- Section header: `// CONTACT`
- Single bordered block (max 600px), monospace copy, email link
- Blinking block cursor after the email address (`.cursor-blink`)

### Links (`#links`)
- Section header: `// LINKS`
- Inline list: GitHub, Twitter/X, Are.na, Instagram
- Hover: underline turns red

---

## Canvas animation (`#bg-canvas`)
Full-viewport fixed canvas, z-index 0, behind all content. Three layered systems drawn each rAF.
**Draw order:** `clearRect → drawTerrain → drawParticles → drawRain`

---

## Audio reactivity system

### Setup
- `AudioContext` + `AnalyserNode` created lazily on first play button click (browser autoplay policy)
- `analyser.fftSize = 256` → 128 frequency bins, `smoothingTimeConstant = 0.82`
- `audioReactive` object holds smoothed values: `{ bass, mid, high }` — all 0..1
- Additional JS lerp on top of Web Audio smoothing: fast attack, slow decay

### Frequency bands
| Band | Bins | Approx Hz |
|---|---|---|
| bass | 0–5 | 0–500 Hz |
| mid | 6–39 | 500–4k Hz |
| high | 41–90 | 4k–10k Hz |

### `audioMod` — scene modulation object
Lives between the audio analyser and the draw functions. Updated every frame in `animate()`.

| Property | Driven by | Effect |
|---|---|---|
| `noiseAmp` | bass | terrain Y displacement amplitude. Idle = `T_NOISE_AMP (240)`, peaks at ~3.4× on hard bass. Lerp attack 0.126, decay same. |
| `timeScale` | mid | multiplier on `dt` fed to `terrainTime`. Idle = 1.0, peaks ~2.26× on full mids. |
| `waveComplexity` | mid | scales the 3 higher-frequency wave layers (indices 2–4) in the terrain formula. Idle = 1.0, peaks ~2.75×. |
| `meshAlpha` | high | terrain grid `globalAlpha`. Idle = 0.17, peaks ~0.69 on bright transients. |
| `particleBoost` | high | multiplier on particle `finalAlpha`. |
| `rainBoost` | — | fixed at 1.0 (rain is intentionally left untouched) |

---

## Layer 1 — Terrain mesh

### Projection
True 90° overhead (camera looking dead down):
```
ry    = -z3
rz    = y3 + T_DEPTH
scale = T_FOV / (T_FOV + rz)
sx    = cx + x3 * scale
sy    = cy + ry * scale
```

### Key constants
| Constant | Value | Purpose |
|---|---|---|
| `T_FOV` | 620 | perspective focal length |
| `T_DEPTH` | 520 | base depth — lower = more dramatic scale variation from noise |
| `T_NOISE_AMP` | 240 | Y displacement amplitude (base — audio scales this at runtime) |
| `T_NOISE_SCALE` | 0.0024 | absolute world-space noise frequency |
| `T_CELL_SIZE` | 44 | world units per cell |
| `T_BLEED` | 1.25 | grid extends 25% past each viewport edge |
| `MAX_GRID_X / Z` | 140 / 110 | performance caps |

### Viewport-responsive cell count
```js
const inv   = (T_FOV + T_DEPTH) / T_FOV;
const halfX = (W / 2) * inv * T_BLEED;
const halfZ = (H / 2) * inv * T_BLEED;
const gridX = Math.min(Math.ceil((halfX * 2) / T_CELL_SIZE), MAX_GRID_X);
const gridZ = Math.min(Math.ceil((halfZ * 2) / T_CELL_SIZE), MAX_GRID_Z);
```
Larger windows → more cells, same apparent square size everywhere.

### Height formula — 5-layer sine wave sum + simplex roughness
```js
const wc = audioMod.waveComplexity;   // mid-driven, scales layers 3-5
const h =
  Math.sin(2.10 * ( nx + 0      ) + t * 0.85) * 0.40 +        // layer 1 — base, unscaled
  Math.sin(1.65 * ( nx*.707 + nz*.707) + t * 0.65) * 0.30 +   // layer 2 — diagonal, unscaled
  Math.sin(2.75 * (-nx*.500 + nz*.866) + t * 1.05) * 0.18 * wc +  // layer 3 — scaled by mids
  Math.sin(1.30 * (-nx*.940 - nz*.342) + t * 0.55) * 0.12 * wc +  // layer 4 — scaled by mids
  Math.sin(3.40 * ( nx*.174 - nz*.985) + t * 1.35) * 0.06 * wc;   // layer 5 — scaled by mids

const rough = Noise.noise2(nx * 1.6 + t*0.18, nz * 1.6 + t*0.14) * 0.13;
const y3    = (h + rough) * audioMod.noiseAmp;
```
Noise coords are absolute world units — larger screens reveal more terrain variation, pattern doesn't stretch.

### Draw
`strokeStyle = '#ffffff'`, `globalAlpha = audioMod.meshAlpha` (0.17 idle, flashes on highs), `lineWidth = 0.3`.
Draws all horizontal lines first, then all vertical lines.

---

## Layer 2 — Floating particle field

**Separate coordinate space from terrain** — uses `project()` not `projectTerrain()`. Do not mix.

| Constant | Value |
|---|---|
| `PARTICLE_COUNT` | 400 |
| `SPREAD_X / Z` | 900 |
| `Z_NEAR / Z_FAR` | 50 / 950 |

### Particle properties
```js
{
  x3, y3, z3,           // 3D position
  vx, vy, vz,           // velocity
  trail:    ~12%,       // emits a short white trail
  reactive: ~35%,       // pulses brightness with bass/mid
  pulse:    0           // lerped pulse value 0..1
}
```

### Size & brightness
- `radius = 1.2 + depthT * 2.4` (bumped up from original — current intentional baseline)
- Depth-based alpha + centre proximity glow boost
- Reactive particles: `pulse` driven by `bass*5.0 + mid*2.0`, threshold at 0.15 (below = decays to 0)
- Pulse lerp: attack 0.4, decay 0.07
- Pulse adds up to +0.55 to `finalAlpha` on reactive particles; size is not modulated

---

## Layer 3 — Rain

200 purely 2D streaks. `RAIN_WIND = 0.09` rad (~5° lean). Intentionally not audio-reactive.
- `layer` (0=far, 1=near): speed `4 + layer*9` px/frame, length `6 + layer*20`, opacity `0.05 + layer*0.24`
- Rebuilt on `resize`.

---

## Music player

IIFE in script. Playlist: `Music/chromedream.mp3`, `Music/quietpulse.mp3`.
- `<audio>` element, volume 1, no autoplay (browser policy)
- Play/pause toggle button (`▶` / `▐▐`), seek slider, time display, track label
- AudioContext initialised on first play button click — **this is intentional and must stay**
- Seek slider fill updated via JS `linear-gradient` to show playback position

**Music folder case sensitivity**: on Linux servers (GitHub Pages etc.) `Music/` must match exactly.
GitHub file size limit is 100MB — if tracks exceed this, host externally and update `src` paths.

---

## Noise implementation
Inline 2D simplex noise (~50 lines), no external library. Seeded with `42`.
API: `Noise.noise2(x, y)` → `[-1, 1]`.

---

## Text animations

### Typewriter (`#hero-text`)
IIFE. Types `'Phantomra.in'` at 145ms/char after 420ms delay.
Cursor: `.typing` (solid) → `.done` (blinking CSS animation).
On completion → immediately calls `startGlitch(target, ...)`.

### Text glitch (`startGlitch(el, opts)`)
Reusable. Snapshots `el.textContent` as canonical truth.
Picks N random `[a-zA-Z0-9]` positions (punctuation/spaces never touched).
Flickers random chars from `GLITCH_CHARS` pool, then restores. Schedules recursively.
```
GLITCH_CHARS = 'ABCDEFGHIJKLNOPQRSTUVXYZ0123456789!#%&<>?'
// M, W, @, [], {} excluded — too wide, caused line-wrap in proportional title font
```
- **Title** (`#hero-text`): 2 chars, 3 flickers × 55ms, every 3–7s
- **Subtitle** (`.hero-subtitle`): 2 chars, 3 flickers × 75ms, every 4.5–9.5s, starts after 2200ms delay

### Scroll fade-in
`IntersectionObserver` (threshold 0.1) adds `.visible` to `#work`, `#contact`, `#links`.
CSS: `opacity 0→1`, `translateY 24px→0`, 0.7s ease.

### Live clock
`setInterval` 1s → `#status-clock` → format `MM/DD::HHMM+SS`.

---

## Critical rules — never break these
- **Never** alter `project()`, `SPREAD_X/Z`, `Z_NEAR`, `Z_FAR`, `TERRAIN_Y_BASE` — particles depend on them
- `projectTerrain()` and `project()` are separate coordinate spaces by design — do not mix
- Red (`#ff2222`) used only on CTA hover + card hover borders — nowhere else
- All headlines: Space Grotesk. Everything else: Space Mono. No exceptions
- No JS frameworks, no Three.js, no build tools — keep it that way
- `white-space: nowrap` on `.hero-title` must stay — prevents glitch reflow
- AudioContext must be initialised lazily on user gesture (play button) — never on page load
- Rain is intentionally not audio-reactive — do not change this
