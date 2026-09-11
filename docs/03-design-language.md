# 03 — Design Language: making the screen feel like a book

## Working order

Build it **ugly and correct first**. The book aesthetic is Milestone 9, after the engine
works. This is not an arbitrary ordering: styling a sheet whose numbers are still moving
means restyling it repeatedly, and an early commitment to a visual structure will quietly
distort your data model to match the layout. Structure follows logic, decoration follows
structure.

## Start with a moodboard, end with tokens

Collect your Pinterest references into a folder. Then, from them, extract a **fixed token
set** — the specific colours, sizes and spacings you will actually use. Everything after that
references tokens only. No raw hex codes anywhere else in the stylesheet, ever.

```css
:root {
  /* ink */
  --ink:          #2b2118;   /* printed rules text  */
  --ink-faded:    #6b5a48;   /* secondary, captions */
  --ink-hand:     #1c3a5e;   /* PLAYER-ENTERED values — deliberately a different ink  */
  --rubric:       #8c2f22;   /* red rubrication for headings, à la an illuminated MS  */
  --gilt:         #b08d4f;

  /* substrate */
  --parchment:    #ece0c8;
  --parchment-lo: #ddcdae;   /* shading toward the gutter and edges */
  --rule:         #b9a888;   /* hairlines */

  /* type */
  --font-display: 'Cinzel', 'Trajan Pro', serif;
  --font-body:    'EB Garamond', 'Libre Baskerville', Georgia, serif;
  --font-hand:    'Caveat', 'Architects Daughter', cursive;

  /* metrics */
  --page-w: 46rem;  --page-h: 62rem;  --gutter: 3rem;
}
```

The `--ink-hand` token carries real weight: **printed values and player values look
different**. Computed numbers render in the book's typeface as though printed at the press;
things you chose render in ink, as though written in. It communicates what the sheet is doing
for you without a single word of explanation.

## Techniques, in build order

### 1. The substrate
Layer three backgrounds on the page element: a flat parchment colour, an SVG `feTurbulence`
fractal-noise texture as an inline data URI at low opacity, and a `radial-gradient` vignette.
Optionally a stain layer at `mix-blend-mode: multiply`. Keep the noise subtle — it must not
compete with text at body size.

### 2. The spread
CSS Grid, two `.page` elements either side of a `.gutter`. The gutter is sold entirely by an
`inset box-shadow` gradient on the inner edge of each page, darkening toward the spine. Add
`perspective` on the container so the flip in step 5 has somewhere to happen.

### 3. The paper block
Stacked pseudo-elements, or a `repeating-linear-gradient` on the outer edges, to suggest the
thickness of pages beneath. A slight `border-radius` on the outer corners only.

### 4. Typography
This is where most of the "book" actually comes from — more than any texture.
`font-feature-settings` for old-style figures, small caps and ligatures. Drop caps via
`::first-letter` or `initial-letter`. Generous leading, a measure of 60–75 characters,
`text-align: justify` with hyphenation on body prose only (never on tables). Section headings
in `--rubric`.

### 5. Page turn
`transform: rotateY()` with `transform-style: preserve-3d` and `backface-visibility: hidden`
on a two-faced leaf element. CSS gets you a convincing *hard fold*; a genuine paper *curl*
requires canvas or WebGL and is not worth it. Gate the whole thing behind
`prefers-reduced-motion` and make the fallback a clean cross-fade.

### 6. Fields that look written, not typed
```css
.sheet input {
  background: transparent; border: 0; border-bottom: 1px dotted var(--rule);
  font-family: var(--font-hand); color: var(--ink-hand); font-size: 1.05em;
}
.sheet input:focus-visible { outline: 2px solid var(--gilt); outline-offset: 2px; }
```
Removing an input's border removes its affordance, so the focus ring becomes load-bearing.
Never ship a field you cannot see when tabbing to it.

## The three constraints that will bite you

**Contrast.** Brown ink on parchment is where this aesthetic naturally lands and it will fail
WCAG AA at body sizes. Because every colour is a token, a `[data-theme="vellum-high"]` block
that redefines six variables gives you a high-legibility mode for anyone who needs it — and
for reading a sheet on a phone in a dim pub, which is most actual play. Build the toggle in
Milestone 9, not as an afterthought.

**Responsive.** The two-page spread is dead below roughly 900px. Don't shrink it — swap it.
Below that breakpoint: one page, tab navigation, no flip. Design the mobile layout as its own
thing rather than as a squeezed book.

**Print.** A `@media print` stylesheet that goes flat and high-contrast: no textures, no
vignette, black on white, explicit `page-break-inside: avoid` on stat blocks. Browser
print-to-PDF is your PDF export; you do not need a PDF library.

## Accessibility, concretely

- Real `<label>` for every field. The visual design hides a lot of structure; the accessibility
  tree must keep it.
- The sheet is a document, so use semantic headings and `<table>` for tabular data. Skills are
  a table. Attack lines are a table.
- Provenance popovers ("why is my AC 18?") must be keyboard reachable, not hover-only.
- Test once with the browser zoomed to 200%. Fixed-size book pages fail this if the metrics
  are in `px` rather than `rem`.
