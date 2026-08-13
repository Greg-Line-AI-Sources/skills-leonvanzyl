# Deliverables: Manifest, Technical Contract, Construction Maths

## SVG technical contract (every file)

- Pure vector: no filters, no blurs, no `<image>`, no live `<text>`, no external fonts.
- Explicit `width` + `height` matching the viewBox, `role="img"`, and a `<title>` — without
  width/height an `<img>` falls back to 300×150 and letterboxes, and Office/Slides mis-size
  viewBox-only SVGs on import.
- Ids prefixed per-asset (inlining two files with duplicate gradient ids corrupts both).
- Counters as real holes via `fill-rule="evenodd"` subpaths, verified in more than one renderer.
- No gradient anywhere unless the brand system explicitly kept one (default: flat colour only).

## The manifest

```
svg/mark/       mark-dark, mark-light, mark-mono-<light>, mark-mono-<dark>, mark-accent
                (tight artboard, the mark's own bounds)
svg/wordmark/   wordmark-dark/-light/-mono-*  — outlined from the font, accent letters
                as their own path group
svg/lockup/     lockup-horizontal-dark/-light, lockup-stacked-dark/-light
svg/icon/       icon-square-*, icon-app-* (plated), icon-avatar-* (round-safe),
                icon-adaptive-foreground/-background/-preview, favicon.svg (+ explicit
                dark/light overrides)
svg/themed/     inline-only variants: currentColor + var(--accent)
png/            favicon 16/32/48 · apple-touch 180 · app 192/512/1024 · adaptive 432 ·
                avatars 400/800 · marks 512/1024 · lockups at 1x/2x widths
README.md       pick-the-right-file table, themed-variant CSS snippet, circular-mask
                warning, favicon <link> block, colour/type/clear-space spec, honest
                known-open-items
guidelines.html generated spec page (below)
```

Package as `<company>-brand-kit/` and zip.

## Construction maths that must not be re-derived by guesswork

**Lockup baseline rule.** The mark's flat foot sits ON the wordmark's baseline (flat edges on
the baseline; only round forms overshoot, by ~1u). Never centre the mark on the wordmark's
bounding box — that is how the source engagement shipped a masthead 2.3× too large. Size the
mark from a *stated geometric relationship*, e.g. "mark height such that its dominant round
form equals cap height exactly"; derive the number, then write the derivation into the spec.

**Wordmark metrics.** Outline with fontTools (`scripts/outline.py`): instantiate the weight
from a variable font, apply GPOS kerning, flip Y, normalise so cap height = 100 units,
baseline = 0. Measure the TRUE ink extremes with BoundsPen (ascender ≠ tallest glyph;
descenders vary) — the viewBox must wrap measured ink, not font metrics.

**Ink radius / round-safe icons (Law 17).** Compute max distance from bbox centre to any ink
point (check every arc extreme and every corner — in the source engagement the farthest ink
was a stem corner, not the bowl). Scale so all ink fits:
- avatars: inside r = 27 of the 64-frame's r = 32
- Android adaptive foreground: inside r = 20 (~66dp guaranteed circle); ship background
  layer + composed preview separately
Centre geometrically for circular crops (the crop is geometric); centre optically
(bbox↔centroid blend, ~0.35 bias) for square containers. Verify by rendering under circle,
square, AND squircle masks before packaging.

**Self-theming favicon.** One `favicon.svg` whose fills come from CSS custom properties in an
embedded `<style>`, with a `prefers-color-scheme: dark` override. A favicon has no
surrounding document, so `currentColor` does not work there — the file must carry its own
theme. Without this, a light-mark favicon on a light tab strip renders ~1:1 contrast and the
only visible pixels are the accent element. Also emit explicit -dark/-light override files
and PNG fallbacks:
```html
<link rel="icon" href="/favicon.svg" type="image/svg+xml">
<link rel="alternate icon" href="/png/favicon-32.png" sizes="32x32">
<link rel="apple-touch-icon" href="/png/apple-touch-icon-180.png">
```

**Themed variants.** Letterform `fill="currentColor"`, accent `fill="var(--accent)"`.
Document loudly: inline-only — as `<img>`/background they render black.

**PNG rendering.** Headless Chrome: `--screenshot --window-size=W,H`, with
`--default-background-color=00000000` for transparency. Wrap the SVG in a minimal HTML page
with the SVG sized via CSS (the explicit width/height attributes will otherwise fight you).

## The generated guidelines page

Generate `guidelines.html` programmatically FROM the shipped assets — inline the real SVGs,
draw the construction plate from the real geometry with dimension lines, compute the stated
measurements from the files. The page then cannot drift from what ships, and every revision
regenerates it for free. Include: the idea (one screen, the phone test verbatim);
construction plate with dimensions; clear-space + minimum sizes (digital px, print mm,
embroidery mm); colour table with computed contrast ratios and the accent-discipline rule;
typography; lockups with the baseline rule stated; variants and when each applies; circular-
mask section with the ink-radius number; misuse list; asset index; and an honest "how it was
made / known limitations" section. Subset the brand font to a data-URI `@font-face` (~9KB
per weight via fontTools subset → woff2) so the page is self-contained. Theme-aware via
`prefers-color-scheme` with `[data-theme]` overrides. Publish as an Artifact when available;
always also ship the file in the kit.

## Verification gates before packaging

1. Round-safe variants rendered under circle/square/squircle masks — nothing clips.
2. Favicon rendered over white, over light-grey (#DEE1E6), over the dark ground at 16px.
3. Lockup: mark foot exactly on baseline; only round forms cross it.
4. Every SVG imports clean (no missing width/height, no duplicate ids).
5. Contact sheet of the full kit — one page, all assets, both grounds — reviewed by eye.
6. README's known-open-items list is current and honest.
