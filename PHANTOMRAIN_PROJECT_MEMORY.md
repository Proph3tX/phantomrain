# PHANTOMRA.IN — Project Memory / Handoff

## Project identity

**PHANTOMRA.IN** is a single-file generative audiovisual portfolio site.

It is not a normal static portfolio. Treat it as a real-time interactive signal environment:

> A monochrome brutalist terminal frame surrounding a living audio-reactive terrain field.

The site should feel like a transmission system, not a conventional landing page.

---

## Current implementation state

### Source of truth

The current source of truth is the latest working `index.html` derived from:

`Index_13_accent_mesh_reactive_v2_hero_glitch_hue.html`

The previous memory file should be ignored except as historical context.

### Architecture

- Single-file HTML site.
- No framework.
- No build step.
- CSS, HTML, and JavaScript all live inside `index.html`.
- Local testing should be done through a local HTTP server, for example:

```bash
npx serve .
```

AudioContext behavior may not work correctly from direct `file://` loading.

---

## Visual identity

### Base style

- Black background.
- White typography.
- Monochrome brutalist UI.
- ASCII / terminal / signal-system aesthetic.
- Thin chrome frame around viewport.
- Top and bottom HUD-style bars.
- Large hero wordmark: `PHANTOMRA.IN`.
- Red-adjacent accent system, now hue-adjustable.

### Fonts

- `Space Grotesk` for large hero/title typography.
- `Space Mono` for UI, labels, status text, and body detail.

### Mood

The target is:

- stark
- precise
- reactive
- spatial
- low-poly / wireframe-adjacent
- audiovisual
- more “signal environment” than “portfolio template”

Avoid making it look like a generic SaaS landing page.

---

## Core page structure

The page currently contains:

1. Fixed background canvas: `#bg-canvas`
2. Fixed viewport frame: `#frame`
3. Top chrome bar:
   - menu glyph
   - PHANTOMRA.IN label
   - window glyph
   - `TUNE` button
4. Signal tuning panel
5. Bottom chrome/status bar:
   - copyright
   - center coordinate/status label
   - music player
   - signal level / volume
   - clock
6. Main content:
   - Hero
   - Work
   - Contact
   - Links

Do not restructure these unless absolutely necessary.

---

## Canvas render pipeline

The background canvas render order is:

```text
clearRect
→ drawTerrain()
→ drawParticles()
→ drawRain()
```

This order is important.

The terrain mesh is the foundation. Particles sit above it. Rain is a thin overlay.

Do not merge terrain, particles, and rain into one system.

---

## Terrain mesh system

### Concept

The mesh is a rolling procedural terrain / signal field.

It should look like a moving landscape or wave surface, not a static grid and not a twitching equalizer.

The correct behavior after the latest working patch is:

> Rolling landscape first, music-reactive second.

### Important lesson from recent iteration

A prior patch made the mesh highly reactive, but it caused the terrain to “twitch in place” instead of sliding/rolling as waves.

That was fixed in v2 by restoring the old rolling sine-wave feel as the base behavior and layering audio reactivity more gently on top.

### Preserve this behavior

Future mesh edits should keep:

- continuous rolling movement
- traveling wave feeling
- landscape-like drift
- music-reactive height/energy changes
- no aggressive per-wave timing modulation that makes it jitter in place

### Audio-reactive mesh controls

The latest working version includes additional tuning related to mesh expressiveness, including:

- `meshMotion`
- `meshRipple`

These should enhance the rolling terrain without overpowering it.

The mesh should show the music’s movement through:

- height changes
- line intensity
- ripple emphasis
- subtle complexity changes
- smoother rolling modulation

It should not behave like a visualizer that merely jitters to transients.

---

## Audio reactivity

### Current design intent

Audio should drive the visual system as “signal energy.”

It affects:

- terrain amplitude
- terrain motion/ripple
- mesh alpha
- mesh line width
- particle brightness
- particle movement/pulses
- hero dot energy
- CTA energy
- signal core influence

### Important implementation note

The site uses Web Audio analysis, including smoothed frequency data and derived band/peak/flux-style signals.

A recent improvement made the mesh more reactive by detecting movement in the audio, not only raw loudness. However, too much transient coupling makes the mesh twitch.

When modifying audio response:

- keep transient response visible
- keep rolling motion dominant
- avoid excessive analyser smoothing
- avoid excessive direct modulation of individual sine wave phase/speed
- prefer gentle additive modulation over replacing the base terrain motion

---

## Accent hue system

### Current state

The accent color is now controlled through a hue slider in the Signal Tuning panel.

The original accent was red. The hue control should stay red-adjacent rather than full rainbow.

The accent system drives:

- terrain mesh accent glow
- particle fill
- particle trails
- hero dot
- CTA glow
- tuning UI accent details
- source/player UI accents
- volume slider fill
- hero glitch text
- subtitle / below-hero accent glitch effects

### Critical rule

All accent color should come from the shared hue/accent system.

Do not reintroduce hardcoded red values such as:

```css
#ff2222
rgb(255,34,34)
rgba(255,34,34,...)
rgba(239,11,11,...)
```

unless they are part of a deliberate fallback that still tracks the accent system.

### Preferred approach

Use the existing accent helper/system from the latest working file.

Expected concepts:

```js
tuning.accentHue
accentHsla(...)
```

and CSS variables such as:

```css
--accent-hue
--accent
```

### Hue range

Keep the hue slider constrained to a red-adjacent range.

Good range:

```js
min: -25
max: 35
```

This allows:

- crimson
- ember
- blood red
- red-orange
- magenta-red

Avoid full-spectrum rainbow color controls. That breaks the identity.

---

## Signal tuning system

### Purpose

The tuning system exists so the visual field can be adjusted live without repeatedly editing constants by hand.

### Structure

The tuning system is based on:

- `DEFAULT_TUNING`
- `TUNING_CONTROLS`
- runtime `tuning`
- localStorage persistence
- export textarea
- reset/copy buttons

### Important rules

- Add new tunable values through `DEFAULT_TUNING` and `TUNING_CONTROLS`.
- Avoid creating one-off UI controls outside the tuning system.
- Keep tuning labels short and readable.
- Keep controls meaningful; do not expose every internal constant.
- If a value affects the site’s identity, it probably belongs in tuning.
- If a value is purely structural/projection math, it probably should not be exposed.

### Export behavior

The current export behavior may still be JSON-like depending on the exact file state.

A future improvement could make the export textarea output paste-ready JS:

```js
const DEFAULT_TUNING = Object.freeze({
  particleCount: 230,
  accentHue: 0,
});
```

This is useful, but not required unless actively requested.

---

## Hero system

### Current behavior

The hero wordmark types/glitches into:

```text
PHANTOMRA.IN
```

The dot is special and should remain a separate accent-reactive element.

The hero dot responds to signal energy and follows the hue slider.

The glitching hero text and below-hero/subtitle accent effects now also follow the hue slider.

### Important rule

Do not let glitch characters create line wrapping.

The hero title should remain on one line.

The CSS currently protects this through nowrap behavior. Preserve that.

---

## Music player

### Current behavior

The bottom chrome includes a small music player with:

- source selector
- skip button
- play/pause
- track label
- seek slider
- time display
- signal level / volume control

The player is part of the site identity. It is not just utility UI.

### Audio source role

Music is used to drive the site’s signal system. The visuals should still look good when music is not playing, but the full experience assumes audio is active.

---

## Particle system

### Current behavior

Particles float in projected 3D space above the terrain.

They respond to audio through:

- brightness
- pulse
- burst
- subtle motion
- signal core push near the hero/dot

Particles should remain visually tied to the mesh and hero dot through the shared accent hue.

Do not let particles become a separate color system.

---

## Rain system

### Current behavior

Rain is a 2D overlay of angled streaks.

It should stay subtle.

Rain adds atmosphere but should not dominate the mesh or particles.

---

## Development workflow

### Primary instruction

Make minimal, surgical edits.

This file is a dense single-file interactive system. Large rewrites are risky.

### Recommended workflow

1. Start from the latest working `index.html`.
2. Identify the exact function/constant/control being changed.
3. Patch only that area.
4. Test visually in browser.
5. Confirm:
   - no JS errors
   - audio still plays
   - tuning panel still opens
   - localStorage tuning still works
   - hue slider still controls all accent areas
   - mesh still rolls as a landscape

### Avoid

- broad refactors
- framework migration
- splitting files unless explicitly requested
- rewriting projection math
- replacing the render loop
- making the mesh a generic equalizer
- adding unnecessary UI panels
- adding full rainbow color controls
- hardcoding accent colors

---

## Known successful recent changes

### 1. Mesh reactivity v2

The mesh was made more reactive while preserving rolling landscape movement.

Key outcome:

> More song-responsive, but still wave-like and sliding.

This v2 behavior should be preserved.

### 2. Accent hue slider

A hue slider was added to Signal Tuning.

It controls the red-adjacent accent color across the whole site.

### 3. Hero glitch hue fix

The final missed hardcoded color was in the hero glitch text.

That was fixed so the glitch text now follows the hue slider too.

---

## Design rules for future changes

### Good changes

A change is probably good if it:

- makes the site feel more unified
- makes the music feel more physically present
- preserves the monochrome brutalist identity
- enhances signal/terrain/mesh coherence
- adds a small amount of control without clutter
- keeps the page mysterious but usable

### Bad changes

A change is probably bad if it:

- makes the site look like a generic music visualizer
- makes the mesh twitch instead of roll
- adds colorful UI unrelated to the core accent
- exposes too many controls
- breaks the single-file simplicity
- makes the site feel like a settings app
- reduces the hero impact
- hides the mesh behind too much particle/rain noise

---

## Future improvement ideas

These are not requirements.

### Safe / likely worthwhile

- Add a “Copy DEFAULT_TUNING” export mode that outputs paste-ready JS.
- Add a small import/paste tuning feature to restore saved tuning values.
- Add a preset selector with 2–3 identities:
  - CRIMSON
  - EMBER
  - MAGENTA
- Add a reduced-motion fallback for accessibility.
- Add a mobile-specific tuning cap so the canvas remains performant.
- Add a tiny FPS/performance debug toggle hidden behind a key command.
- Improve source/track metadata if real hosted music files are used.

### Riskier

- More complex mesh shading.
- More aggressive beat detection.
- Mouse interaction with terrain.
- Scroll-position-driven mesh changes.
- Multi-layer terrain passes.
- Post-processing/glow effects.

Riskier ideas may look good, but they can easily break the clean brutalist identity or hurt performance.

---

## Current north star

The site should feel like:

> A black-and-white brutalist transmission terminal where music wakes the terrain, colors the signal, and turns the page into a living phantom field.

Keep that feeling intact.
