---
name: style-guide
description: >
  Generate a standalone brand style-guide HTML page from any website URL. Invoke with
  /style-guide <url> — the skill scrapes the site, extracts its brand system (logo, colors,
  typography, voice), and produces a single self-contained HTML file in the proven Reblis
  style-guide layout: full-bleed gradient header with inline SVG logo, sticky anchor nav,
  numbered sections, swatch grids, shade ramps, type specimens, and Lucide.dev icons.
  Trigger when the user says "style guide", "brand guide", "make a style guide for
  <site>", or invokes /style-guide. Authored by Reblis.com.
---

# Style Guide Generator — by Reblis

Generate a **standalone brand style-guide HTML file** from a website URL.

Usage: `/style-guide <url>` (e.g. `/style-guide https://patientsfirst.org`)

The canonical example of the finished product is `template-example.html` in this skill
directory (the Shalom Estetica brand guide, generated from shalomestetica.co). **Read it
before generating** — it shows the layout, CSS architecture, and component set to
reproduce with the target site's brand.

## Hard Rules (non-negotiable)

1. **Standalone file.** One HTML file, zero external asset dependencies. The only allowed
   external reference is Google Fonts. Everything else — logo, icons, swatches — is inline.
2. **Logo is inline SVG code** on the page. Never `<img src>`, never base64 raster.
3. **Sticky anchor menu.** A frosted sticky nav (`position: sticky; top: 0`) with uppercase
   anchor links to every numbered section.
4. **Icons are always Lucide** (https://lucide.dev/icons/). Never HTML entities, emoji, or
   other icon sets. Fetch exact official SVG code — do not recall paths from memory:
   `curl -sfL "https://unpkg.com/lucide-static@latest/icons/<name>.svg"`
   (the `-L` matters — unpkg 302-redirects `@latest`). Strip the license comment, collapse
   to one line, inline it. Lucide ships 24×24 viewBox with `stroke="currentColor"`, so
   color comes from the parent's CSS `color` and size from CSS width/height on the svg.
5. **Output location:** write the file where the user asks; default to the relevant
   project folder as `style-guide.html`. Never stream the HTML into chat.
6. **No header glow.** Never add a `::before` radial-gradient glow blob (e.g.
   `radial-gradient(circle, rgba(...), transparent 68%)`) to the header band — on any
   guide, for any brand. The header is a flat color or linear gradient only; texture
   comes from the brand's own ornaments (torn edges, accent rules), never from a glow.

## Step 1 — Scrape the brand from the URL

Fetch the page (curl with a real browser UA; fall back to scrapling stealth if blocked).
Extract:

- **Logo**: look for inline `<svg>` in the header, or a linked `.svg` (check `<img src>`,
  `<link rel=icon>`, og:image). Prefer the horizontal lockup. If no usable SVG exists,
  note it to the user and fall back to a **wordmark set in the brand's own H1/display
  font** — the heaviest loaded weight, white, with the site's H1 letter-spacing, in the
  logo slot. Never substitute an off-brand face (no Pacifico or other script fallback);
  the brand's display face is what makes the wordmark read as a mark.
- **Colors**: CSS custom properties in stylesheets, then computed fills on buttons/links/
  headers. Identify primary / secondary / dark anchor / surface tints. Build 5-step shade
  ramps (81% / 62% / 43% / 24% / 5% mixes toward white) when the brand doesn't define its own.
- **Typography**: Google Fonts `<link>`s and `font-family` declarations. Identify the
  display face and body face (may be one face — weight carries hierarchy then).
- **Radius**: the site's card/surface `border-radius`. Set a single `--radius` token and
  drive every card off it (`border-radius: var(--radius)`); leave pills (`999px`) and
  circles (`50%`) alone. A flat brand (Stratomation, 0) must render sharp corners; a
  rounded brand (Shalom 16px, Ricarte 8px) keeps its curve. Don't assume a value.
- **Voice & content**: tagline, mission/about copy, nav labels, CTA verbs, audiences.
  Derive voice traits and do/don't rows from how the site actually writes.
- **Re-probe, don't assume**: verify what the live HTML actually serves before trusting
  any pattern from a previous site.

## Step 2 — Sanitize the logo SVG

SVGs exported from Illustrator carry `<defs><style>.cls-1{fill:#...}</style></defs>`.
Class-styled inline SVGs **leak styles across the whole document** — two logos with the
same `.cls-1` class will both render the last-defined color. Always:

1. Strip the `<?xml ?>` declaration and the entire `<defs><style>...</style></defs>` block.
2. Replace every `class="cls-N"` with `fill="currentColor"`.
3. Collapse whitespace to a single line.
4. Add `role="img" aria-label="<Brand>"` and set color via `style="color:#FFF"` (or CSS).
5. Size by **height** (e.g. `height: 40px; width: auto`) so lockup and icon variants of
   the same mark render at matching heights.

## Step 3 — Build the page

Reproduce the structure of `template-example.html`, re-tokenized to the scraped brand:

- `:root` design tokens: brand palette, surface tints, neutral text stack (ink/slate/mid),
  rules, functional green/alert, font stacks.
- **Header**: full-bleed gradient band (brand primary → dark anchor), thin accent rule on
  the bottom edge (`::after`), kicker line ("Brand Style Guide · <year> Edition"), the
  **inline SVG logo as the title** (no `<h1>` text duplicate of the logo), lede paragraph
  (weight 400, max-width 770px), uppercase meta line (tagline · typeface).
- **Sticky nav**: brand-name chip left, uppercase anchor links right, blur backdrop.
- **Numbered sections** (adapt to what the brand has; typical order):
  `01 · Foundation` (mission card, tagline banner, position callout, audiences, value
  pillar-cards with Lucide icons in dark chips) → `02 · Voice & Tone` (trait cards with
  do/don't rows, voice-spectrum bars) → `03 · Tone by Audience` (quote cards) →
  `04 · Language` (always-say/never-say table, phrasing cards, capitalization callout) →
  `05 · Logo` (variant tiles) → `06 · Color` (swatch cards, shade ramps, gradients) →
  `07 · Typography` (type rows, scale bars, rules callout). Add a **`Layout`** section
  whenever the site has a defined container width + responsive gutters/section rhythm
  (see the box-model rule below) — slot it among the foundational sections (typically
  right after Typography).
- **Specimen rows are vertically centered.** Any row that places a label beside a type
  sample (type-specimen rows, type-scale rows) uses `align-items: center`, never
  `baseline` — the small label sits centered against the tall sample, not dropped to its
  baseline.
- **Layout = interactive box model, not bar charts.** Represent the layout system as a
  Chrome-DevTools-style nested box, never as standalone width bars. Three nested bands:
  **margin** (outer, a 45° hatch of `--ink` at ~6% — the auto-centering space), **padding**
  (the brand's accent color at ~9% with a solid accent edge — this band *is* the gutter
  left/right and the section rhythm top/bottom), and **content** (a paper column edged in
  the brand's primary/anchor color — the container, with its max-width + prose cap noted).
  Edge numbers (gutter px on L/R, rhythm px on T/B, `auto` on the margin) sit centered on
  each edge. A **breakpoint toggle** (Mobile / Tablet / Desktop) styled in the *site's own
  button look* — reuse the scraped button class; active = accent fill — swaps the numbers
  and animates the padding via a few lines of JS. All values (container, 3 gutters, 3
  rhythms, prose cap) come from the scraped CSS; colors come from the brand tokens, so the
  same component reads correctly on any site. See `template-example.html` `#layout`.
- **Labels under tiles**: logo variants and gradient cards get an uppercase label *below*
  the tile (outside the colored box) in one consistent dark grey (`--slate`), never dim
  white inside the tile. Gradient labels follow the "Light / Full / Dark Gradient" pattern.
- **Grid association**: the primary color cards and the shade ramps use the *same* grid
  (`repeat(auto-fit, minmax(220px, 1fr))` — `auto-fit`, NOT `auto-fill`) so each primary
  sits directly above its own ramp.
- **Footer**: brand name + guide title left, monospace stamp right. No tool/source
  attributions in the page itself unless the user asks.
- **Generate programmatically**: the shade ramps, swatch grids, and gradient cards are
  computed data (ramp math, luminance-picked text color) — build the HTML with a small
  Python script and write the file in one shot, as the palette-guide and font-guide
  skills do; don't hand-write ramp steps and gradient cards.

## Step 4 — Verify before claiming done

Screenshot the file headlessly and **look at the images** — header, every section, footer.
Fonts need a `--virtual-time-budget` so Google Fonts actually load before capture (a
specimen rendering in system sans = the webfont didn't load):

```bash
timeout 90 chromium --headless --disable-gpu --no-sandbox \
  --screenshot=check.png --window-size=1400,8400 --hide-scrollbars \
  --virtual-time-budget=8000 "file:///path/to/style-guide.html"
convert check.png -crop 1400x1000+0+<offset> slice.png   # crop slices to inspect
```

Snap-packaged chromium gotcha: it cannot read arbitrary home paths and has a **private
/tmp** — copy the HTML into `~/snap/chromium/common/` first, render it from there, and
write screenshots there too. Verify standalone-ness by rendering a copy with no sibling
assets folder: the logo and icons must still appear. Clean up temp copies afterwards.
