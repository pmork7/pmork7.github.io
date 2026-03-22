# Cellular Automata Hero Visualization — Design Spec

## Overview

Add an interactive Game of Life cellular automata visualization to the hero section of the personal portfolio page. The visualization sits in a contained canvas element to the right of the existing hero text, creating a side-by-side layout.

## Layout

- **Desktop (>600px):** Flexbox row, 50/50 split. Left side: existing hero text (name, tagline, CTA buttons). Right side: canvas + rule selector.
- **Mobile (<=600px):** Stacks vertically — text on top, canvas below. Canvas goes full-width with 16:9 aspect ratio.
- Canvas has a white background (#fff) with black (#000) cells, rounded corners (8px border-radius).
- Rule selector dropdown sits directly below the canvas.

## Simulation Engine

- 2D boolean grid sized dynamically to fill the canvas at ~8px cell size (~40-50 columns on desktop).
- Simulation ticks at ~10 FPS using `requestAnimationFrame` with frame throttle.
- Each tick applies the current Birth/Survival rule to compute the next generation using standard Moore neighborhood (8 surrounding cells).
- On page load, ~25% of cells are randomly seeded as alive.
- Grid state resets on window resize since dimensions change.

## Rule System

Rules use standard B/S notation for Life-like cellular automata. Preset dropdown includes:

| Name | Rule | Character |
|------|------|-----------|
| Conway's Game of Life | B3/S23 | Classic, balanced |
| HighLife | B36/S23 | Replicators |
| Day & Night | B3678/S34678 | Symmetric, dense |
| Seeds | B2/S | Explosive, chaotic |
| Diamoeba | B35678/S5678 | Amoeba-like blobs |
| Morley (Move) | B368/S245 | Slow, structured |

When the user changes the rule via the dropdown, the current grid state is preserved — only the evolution logic changes.

## Hover Interaction & Color Effect

- Canvas tracks mouse position and maps to grid coordinates.
- When the mouse hovers over or near live cells, those cells within a radius of ~3-4 cells get assigned a hue based on their angle relative to the cursor, creating a rainbow spectrum radiating outward.
- Each colored cell stores a "color intensity" value (0 to 1) that decays over ~1 second with an ease-out curve.
- At intensity 0: cell renders as black (normal). At intensity 1: cell renders in its assigned HSL color at full saturation.
- Only live cells respond to hover — dead cells are not colored.
- When the mouse leaves the canvas, all color intensities decay naturally.
- On mobile, `touchmove` events trigger the same color effect as mouse hover. `touchend` lets colors decay naturally.

## Accessibility

- The canvas is decorative — mark with `aria-hidden="true"`.
- The rule selector `<select>` gets a proper `<label>` element.

## Rendering Pipeline (per frame)

1. Clear canvas to white (#fff).
2. For each live cell:
   - If color intensity > 0: draw in its HSL color at current intensity.
   - Otherwise: draw in black (#000).
3. Advance simulation state.

## Performance

- Canvas rendering uses filled rectangles only — no paths or gradients.
- Simulation logic is O(rows * cols) per tick, bounded by canvas size (~2000-3000 cells max).
- `requestAnimationFrame` with tick throttle prevents unnecessary spinning.
- Color decay runs in the same render loop — no extra timers.

## Implementation Approach

**Pure Canvas** — all rendering via HTML Canvas 2D API. Rule selector is a standard `<select>` element. Zero external dependencies.

## File Changes

1. **`src/pages/index.astro`** — Restructure hero section HTML to flexbox layout with text left and canvas right. Add `<canvas>` element, `<select>` dropdown for rules, and inline `<script>` containing all simulation logic.
2. **`src/styles/global.css`** — Update `.hero` styles for flexbox layout, add canvas container and rule selector styles, adjust mobile breakpoint for stacked layout.

No new files. No new dependencies.
