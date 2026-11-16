# Micro interactions

A daily practice: one small, highly refined UI micro interaction per day, each added as a cell in a big scrollable grid. The point is craft. Every piece should push one idea further than a typical UI library would, and feel physically right in the hand.

## How the project is built

Plain HTML, CSS and JavaScript. No build step, no framework, no package.json. Everything works by opening `index.html` straight from disk, and deploys as a static site (GitHub Pages or Vercel).

```
index.html                 the grid; reads manifest.js, newest piece first
manifest.js                window.MANIFEST, one entry per piece in creation order
shared/tokens.css          palette, type, cell shape, easing (shared by everything)
_template/index.html       starting point for a new piece
interactions/NNN-slug/     one folder per piece, each a self-contained index.html
IDEAS.md                   backlog of ideas
```

The grid embeds each piece in an iframe at a fixed aspect ratio, so pieces are fully isolated from each other. Each piece's own URL also works as a full-screen view.

## Rules for every piece

- One folder, one `index.html`, created from `_template/index.html`. Inline CSS and JS in that file. The only shared dependency is `../../shared/tokens.css`.
- Use colors only from the tokens. Light and dark mode must both work; if the piece reads token values in JS, re-read them on `prefers-color-scheme` change.
- Centered in the cell, designed for the square cell (roughly 280 to 450 px in the grid) and still good full-screen.
- No text inside the piece unless the text is the interaction itself. Titles live in the grid caption.
- Accessible: real semantic element (button, input, role="switch" etc.), keyboard operable, visible `:focus-visible` ring in `--signal`, `prefers-reduced-motion` respected with a calmer but still satisfying version.
- Performance: animation loops (requestAnimationFrame) must stop when the piece is at rest. Dozens of pieces run at once in the grid.
- Pointer input via Pointer Events with pointer capture; `touch-action: none` only on the element being dragged.
- External libraries only when they earn their place, loaded from a CDN with an exact pinned version. Prefer hand-written physics and canvas/SVG/WebGL.

## Adding a new piece (the daily routine)

1. Pick the next number (three digits, zero padded) and a short kebab-case slug.
2. Copy `_template/index.html` to `interactions/NNN-slug/index.html`.
3. Build the interaction. Iterate on the feel: tune springs, timing, and edge cases (fast flicks, interrupted animations, rapid repeat input, keyboard).
4. Add an entry to the end of `window.MANIFEST` in `manifest.js` with id, slug, title (sentence case), date (YYYY-MM-DD) and a one-sentence note describing what makes it feel good.
5. Tick the idea off in `IDEAS.md` if it came from there, and add a line to the log below.
6. Commit with the message `NNN: Title`.

## Tuning notes and conventions

- Springs: semi-implicit Euler with 4 substeps per frame and dt capped at 1/30 s. Mass 1; describe springs as stiffness k and damping ratio (c = 2 * sqrt(k) * ratio). Released springs around ratio 0.4 for visible character, near 0.8 to 1 while being dragged so the element trails the pointer and feels heavy.
- Squash and stretch preserves rough volume (when one axis shrinks, the other grows).
- Flick velocity: average over the last few pointer samples, and ignore it if the pointer sat still for more than about 80 ms before release.

## Log

- 001 Inertia toggle (2026-11-16): switch with mass. Flicks carry momentum into the knob, which squashes against the end; slow drags lag the finger. Tap gives a small push. Track color follows the knob position.
- 002 Slam button (2026-11-16): 1px outline slab, fill matches the page, drawn as an SVG path so it can bend: the ends hang off the middle on a wobbly spring driven by the middle's acceleration, so they sag when lifted, stand up under the finger and slap down on impact. The "Floppy button" label is split into letters that each follow the bent surface (drop, scale, slope angle). Hover lifts it on a soft spring with a shadow underneath, and three under-layers peel up after it, each on a looser spring at a share of its height, so the stack fans out with a stagger; they can't pass the layer above or the page, so a slam collapses them; press drops it under gravity into a stiff contact spring, so it squashes into the page and the impact speed sets the ripple strength. Ripples travel through a canvas dot grid: crests push dots outward and swell and darken them, wave fronts follow the slab's rounded-rect outline and turn circular as they spread. Held in, the slab dents the page and pulls nearby dots toward it.
