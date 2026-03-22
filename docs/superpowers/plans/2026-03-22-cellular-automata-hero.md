# Cellular Automata Hero Visualization — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add an interactive Game of Life cellular automata visualization to the hero section, displayed in a canvas to the right of the existing hero text, with a rule selector dropdown and rainbow hover effect.

**Architecture:** Pure Canvas 2D API rendering with an inline `<script>` in the Astro page. The hero section becomes a two-column flexbox layout (text left, canvas right) that stacks on mobile. Zero external dependencies.

**Tech Stack:** Astro, vanilla JavaScript, Canvas 2D API, CSS

---

## File Structure

| File | Action | Responsibility |
|------|--------|---------------|
| `src/pages/index.astro` | Modify (lines 39-51) | Hero HTML restructure + inline `<script>` with all simulation logic |
| `src/styles/global.css` | Modify (lines 160-202, 377-380) | Hero flexbox layout, canvas styles, rule selector styles, responsive adjustments |

No new files.

---

### Task 1: Update Hero HTML Structure

**Files:**
- Modify: `src/pages/index.astro:39-51`

- [ ] **Step 1: Replace hero section with two-column layout**

Replace lines 39-51 in `src/pages/index.astro` with:

```html
  <!-- Hero -->
  <section class="hero">
    <div class="container hero-layout">
      <div class="hero-text">
        <h1>Hi, I'm Paul.</h1>
        <p class="tagline">
          Software engineer who likes building cool things.
        </p>
        <div class="hero-cta">
          <a href="/#projects" class="btn btn-primary">See my work</a>
          <a href="/#contact" class="btn btn-ghost">Get in touch</a>
        </div>
      </div>
      <div class="hero-canvas-wrapper">
        <canvas id="gol-canvas" aria-hidden="true"></canvas>
        <div class="gol-controls">
          <label for="gol-rule" class="gol-label">Rule:</label>
          <select id="gol-rule" class="gol-select">
            <option value="B3/S23">Conway's Game of Life (B3/S23)</option>
            <option value="B36/S23">HighLife (B36/S23)</option>
            <option value="B3678/S34678">Day & Night (B3678/S34678)</option>
            <option value="B2/S">Seeds (B2/S)</option>
            <option value="B35678/S5678">Diamoeba (B35678/S5678)</option>
            <option value="B368/S245">Morley (B368/S245)</option>
          </select>
        </div>
      </div>
    </div>
  </section>
```

- [ ] **Step 2: Verify the page loads without errors**

Run: `npm run dev`

Open in browser. Confirm the hero shows text on the left and a blank canvas area on the right (canvas won't render yet — just verify no build errors and the HTML structure is visible in dev tools).

- [ ] **Step 3: Commit**

```bash
git add src/pages/index.astro
git commit -m "feat: restructure hero section to two-column layout with canvas and rule selector"
```

---

### Task 2: Add Hero Layout and Canvas CSS

**Files:**
- Modify: `src/styles/global.css:160-202` (hero styles)
- Modify: `src/styles/global.css:377-380` (responsive media query)

- [ ] **Step 1: Replace existing hero CSS with flexbox layout styles**

Replace lines 160-202 in `src/styles/global.css` (from `/* Hero */` through `.btn-ghost { ... }`) with:

```css
/* Hero */
.hero {
  padding: 5rem 0 4rem;
}

.hero-layout {
  display: flex;
  align-items: center;
  gap: 2rem;
}

.hero-text {
  flex: 1;
  min-width: 0;
}

.hero h1 {
  font-size: 2.25rem;
  margin-bottom: 0.75rem;
}

.hero .tagline {
  font-size: 1.1rem;
  color: var(--color-muted);
  margin-bottom: 2rem;
}

.hero-cta {
  display: inline-flex;
  gap: 1rem;
  flex-wrap: wrap;
}

.btn {
  display: inline-block;
  padding: 0.5rem 1.25rem;
  border-radius: 6px;
  font-size: 0.9rem;
  font-weight: 500;
  transition: opacity 0.15s;
}

.btn:hover { opacity: 0.85; text-decoration: none; }

.btn-primary {
  background: var(--color-accent);
  color: #fff;
}

.btn-ghost {
  border: 1px solid var(--color-border);
  color: var(--color-text);
  background: transparent;
}

/* Game of Life canvas */
.hero-canvas-wrapper {
  flex: 1;
  min-width: 0;
}

#gol-canvas {
  width: 100%;
  aspect-ratio: 4 / 3;
  border-radius: 8px;
  border: 1px solid var(--color-border);
  display: block;
  background: #fff;
}

.gol-controls {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-top: 0.5rem;
}

.gol-label {
  font-size: 0.8rem;
  color: var(--color-muted);
  font-family: var(--font-mono);
}

.gol-select {
  flex: 1;
  font-size: 0.8rem;
  font-family: var(--font-mono);
  padding: 0.3rem 0.5rem;
  border: 1px solid var(--color-border);
  border-radius: 4px;
  background: #fff;
  color: var(--color-text);
  cursor: pointer;
}
```

- [ ] **Step 2: Update responsive media query**

Replace the existing media query block (lines 377-380) with:

```css
/* Responsive */
@media (max-width: 600px) {
  .hero h1 { font-size: 1.75rem; }
  .hero-layout {
    flex-direction: column;
  }
  #gol-canvas {
    aspect-ratio: 16 / 9;
  }
  .post-item { flex-direction: column; gap: 0.1rem; }
}
```

- [ ] **Step 3: Verify layout in browser**

Run: `npm run dev`

Confirm:
- Desktop: text on left, canvas area on right, 50/50 split
- Rule dropdown visible below the canvas
- Resize to <600px: layout stacks vertically, canvas below text

- [ ] **Step 4: Commit**

```bash
git add src/styles/global.css
git commit -m "feat: add hero flexbox layout and canvas styles with responsive stacking"
```

---

### Task 3: Implement Simulation Engine (Grid + Rules + Tick Loop)

**Files:**
- Modify: `src/pages/index.astro` (add `<script>` before closing `</BaseLayout>`)

- [ ] **Step 1: Add inline script with grid initialization and simulation loop**

Add the following `<script>` block just before the closing `</BaseLayout>` tag at the end of `src/pages/index.astro`:

```html
<script>
(function () {
  const canvas = document.getElementById("gol-canvas");
  if (!canvas) return;
  const ctx = canvas.getContext("2d");

  const CELL_SIZE = 8;
  const TICK_INTERVAL = 100; // ~10 FPS
  const INITIAL_DENSITY = 0.25;

  let cols = 0;
  let rows = 0;
  let grid = [];
  let lastTick = 0;

  // --- Rule parsing ---
  const RULES = {
    "B3/S23": { birth: new Set([3]), survival: new Set([2, 3]) },
    "B36/S23": { birth: new Set([3, 6]), survival: new Set([2, 3]) },
    "B3678/S34678": { birth: new Set([3, 6, 7, 8]), survival: new Set([3, 4, 6, 7, 8]) },
    "B2/S": { birth: new Set([2]), survival: new Set() },
    "B35678/S5678": { birth: new Set([3, 5, 6, 7, 8]), survival: new Set([5, 6, 7, 8]) },
    "B368/S245": { birth: new Set([3, 6, 8]), survival: new Set([2, 4, 5]) },
  };

  let currentRule = RULES["B3/S23"];

  // --- Grid setup ---
  function initGrid() {
    const rect = canvas.getBoundingClientRect();
    canvas.width = rect.width * devicePixelRatio;
    canvas.height = rect.height * devicePixelRatio;
    ctx.scale(devicePixelRatio, devicePixelRatio);

    cols = Math.floor(rect.width / CELL_SIZE);
    rows = Math.floor(rect.height / CELL_SIZE);

    grid = [];
    for (let r = 0; r < rows; r++) {
      grid[r] = [];
      for (let c = 0; c < cols; c++) {
        grid[r][c] = Math.random() < INITIAL_DENSITY;
      }
    }
  }

  // --- Simulation step ---
  function countNeighbors(r, c) {
    let count = 0;
    for (let dr = -1; dr <= 1; dr++) {
      for (let dc = -1; dc <= 1; dc++) {
        if (dr === 0 && dc === 0) continue;
        const nr = r + dr;
        const nc = c + dc;
        if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && grid[nr][nc]) {
          count++;
        }
      }
    }
    return count;
  }

  function step() {
    const next = [];
    for (let r = 0; r < rows; r++) {
      next[r] = [];
      for (let c = 0; c < cols; c++) {
        const n = countNeighbors(r, c);
        if (grid[r][c]) {
          next[r][c] = currentRule.survival.has(n);
        } else {
          next[r][c] = currentRule.birth.has(n);
        }
      }
    }
    grid = next;
  }

  // --- Render ---
  function render() {
    const w = canvas.width / devicePixelRatio;
    const h = canvas.height / devicePixelRatio;
    ctx.clearRect(0, 0, w, h);

    for (let r = 0; r < rows; r++) {
      for (let c = 0; c < cols; c++) {
        if (grid[r][c]) {
          ctx.fillStyle = "#000";
          ctx.fillRect(c * CELL_SIZE, r * CELL_SIZE, CELL_SIZE - 1, CELL_SIZE - 1);
        }
      }
    }
  }

  // --- Main loop ---
  function loop(timestamp) {
    if (timestamp - lastTick >= TICK_INTERVAL) {
      step();
      lastTick = timestamp;
    }
    render();
    requestAnimationFrame(loop);
  }

  // --- Rule selector ---
  const ruleSelect = document.getElementById("gol-rule");
  ruleSelect.addEventListener("change", function () {
    currentRule = RULES[this.value];
  });

  // --- Resize handling ---
  let resizeTimer;
  window.addEventListener("resize", function () {
    clearTimeout(resizeTimer);
    resizeTimer = setTimeout(function () {
      ctx.setTransform(1, 0, 0, 1, 0, 0);
      initGrid();
    }, 200);
  });

  // --- Start ---
  initGrid();
  requestAnimationFrame(loop);
})();
</script>
```

- [ ] **Step 2: Verify simulation runs in browser**

Run: `npm run dev`

Confirm:
- Black cells appear on white canvas, evolving each tick (~10 FPS)
- Changing the rule dropdown changes the evolution behavior (try "Seeds" — should be explosive and chaotic)
- Resizing the window resets the grid to fit new canvas size

- [ ] **Step 3: Commit**

```bash
git add src/pages/index.astro
git commit -m "feat: implement Game of Life simulation engine with rule selector"
```

---

### Task 4: Add Rainbow Hover Effect with Fade

**Files:**
- Modify: `src/pages/index.astro` (add hover logic inside the existing `<script>`)

- [ ] **Step 1: Add color state arrays and mouse tracking**

Inside the IIFE in the `<script>`, after the `let lastTick = 0;` line, add:

```javascript
  let colorIntensity = []; // 2D array: 0-1 fade value per cell
  let colorHue = [];       // 2D array: hue (0-360) per cell
  let mouseX = -1;
  let mouseY = -1;
  const COLOR_RADIUS = 3.5;
  const FADE_SPEED = 0.08; // per frame, ease-out (~1s decay at 60fps)
```

- [ ] **Step 2: Initialize color arrays in initGrid**

At the end of the `initGrid()` function, after the grid loop, add:

```javascript
    colorIntensity = [];
    colorHue = [];
    for (let r = 0; r < rows; r++) {
      colorIntensity[r] = new Float32Array(cols);
      colorHue[r] = new Float32Array(cols);
    }
```

- [ ] **Step 3: Add mouse/touch event listeners**

After the resize event listener block, before the `// --- Start ---` comment, add:

```javascript
  // --- Mouse/touch tracking ---
  canvas.addEventListener("mousemove", function (e) {
    const rect = canvas.getBoundingClientRect();
    mouseX = e.clientX - rect.left;
    mouseY = e.clientY - rect.top;
  });

  canvas.addEventListener("mouseleave", function () {
    mouseX = -1;
    mouseY = -1;
  });

  canvas.addEventListener("touchmove", function (e) {
    e.preventDefault();
    const rect = canvas.getBoundingClientRect();
    const touch = e.touches[0];
    mouseX = touch.clientX - rect.left;
    mouseY = touch.clientY - rect.top;
  }, { passive: false });

  canvas.addEventListener("touchend", function () {
    mouseX = -1;
    mouseY = -1;
  });
```

- [ ] **Step 4: Add color update function**

After the mouse/touch event listeners, add:

```javascript
  // --- Color hover effect ---
  function updateColors() {
    const curCol = Math.floor(mouseX / CELL_SIZE);
    const curRow = Math.floor(mouseY / CELL_SIZE);

    for (let r = 0; r < rows; r++) {
      for (let c = 0; c < cols; c++) {
        if (mouseX >= 0 && mouseY >= 0 && grid[r][c]) {
          const dr = r - curRow;
          const dc = c - curCol;
          const dist = Math.sqrt(dr * dr + dc * dc);
          if (dist <= COLOR_RADIUS) {
            const angle = Math.atan2(dr, dc);
            const hue = ((angle / Math.PI) * 180 + 360) % 360;
            colorHue[r][c] = hue;
            colorIntensity[r][c] = 1;
          }
        }
        // Decay
        if (colorIntensity[r][c] > 0) {
          colorIntensity[r][c] *= (1 - FADE_SPEED);
          if (colorIntensity[r][c] < 0.01) colorIntensity[r][c] = 0;
        }
      }
    }
  }
```

- [ ] **Step 5: Update render function to use colors**

Replace the existing `render()` function with:

```javascript
  function render() {
    const w = canvas.width / devicePixelRatio;
    const h = canvas.height / devicePixelRatio;
    ctx.clearRect(0, 0, w, h);

    updateColors();

    for (let r = 0; r < rows; r++) {
      for (let c = 0; c < cols; c++) {
        if (grid[r][c]) {
          const intensity = colorIntensity[r][c];
          if (intensity > 0) {
            const hue = colorHue[r][c];
            ctx.fillStyle = "hsl(" + hue + " 80% " + (intensity * 50) + "%)";
          } else {
            ctx.fillStyle = "#000";
          }
          ctx.fillRect(c * CELL_SIZE, r * CELL_SIZE, CELL_SIZE - 1, CELL_SIZE - 1);
        }
      }
    }
  }
```

The HSL lightness interpolation ensures cells stay solid: at full intensity (lightness 50%) the cell is a vivid hue, and as intensity decays toward 0 the lightness drops to 0% (black).

- [ ] **Step 6: Verify hover effect in browser**

Run: `npm run dev`

Confirm:
- Moving the mouse over live cells causes nearby cells to light up in rainbow colors radiating from the cursor
- Colors fade back to black over ~1 second after moving away
- Moving the mouse off the canvas lets all colors decay
- On mobile view (devtools device emulation): touch-dragging triggers the same effect

- [ ] **Step 7: Commit**

```bash
git add src/pages/index.astro
git commit -m "feat: add rainbow hover effect with color fade on Game of Life canvas"
```

---

### Task 5: Widen Container for Hero Section

**Files:**
- Modify: `src/styles/global.css`

The current `.container` has `max-width: 720px`. The two-column hero layout needs more room for the canvas to breathe. The hero should use a wider max-width while all other sections stay at 720px.

- [ ] **Step 1: Add hero container override**

In `src/styles/global.css`, after the `.hero` padding rule, add:

```css
.hero .container {
  max-width: 960px;
}
```

Since the hero HTML uses `class="container hero-layout"`, this targets only the hero's container.

- [ ] **Step 2: Verify in browser**

Confirm the hero section is wider than the other sections, giving the canvas adequate space, while About/Projects/etc. remain at 720px.

- [ ] **Step 3: Commit**

```bash
git add src/styles/global.css
git commit -m "feat: widen hero container to 960px for two-column layout"
```

---

### Task 6: Final Verification

- [ ] **Step 1: Full desktop walkthrough**

Run: `npm run dev`

Verify all of the following:
- Hero loads with text left, canvas right, simulation running
- Rule dropdown changes evolution behavior
- Hover creates rainbow color effect that fades back to black
- Scrolling past hero to other sections — everything still looks normal
- No console errors

- [ ] **Step 2: Mobile walkthrough**

In browser devtools, toggle device emulation (e.g. iPhone 14):
- Hero stacks vertically — text on top, canvas below with 16:9 ratio
- Touch-drag on canvas triggers color effect
- Rule selector is usable on small screen

- [ ] **Step 3: Build check**

Run: `npm run build`

Confirm build completes without errors.

- [ ] **Step 4: Commit any final adjustments**

If any tweaks were needed during verification, commit them:

```bash
git add src/pages/index.astro src/styles/global.css
git commit -m "fix: final adjustments from verification walkthrough"
```
