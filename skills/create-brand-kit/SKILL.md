---
name: create-brand-kit
description: >
  Run a multi-agent adversarial brand-identity process from any input — a company website,
  a written brief, or just a name and concept — through to a complete packaged brand kit:
  logomark, outlined wordmark, lockups, round-safe icons, self-theming favicon, PNG rasters,
  a generated guidelines page, and a README. Use this whenever the user wants a logo, a brand,
  a visual identity, a rebrand, a logomark, brand assets, a favicon/icon set, or says things
  like "design a logo for my company", "we need branding", "make us a brand kit", or shares a
  company site asking for identity work. Even if they only ask for "a logo", use this skill —
  a logo that hasn't survived adversarial review and doesn't ship with its variants is not done.
---

# create-brand-kit

Produce a professional brand identity the way a AAA studio would: divergent concepts,
devil's-advocate critics, judge panels, hand refinement, adversarial verification, then a
complete production kit. This process was derived from a real engagement in which **the first
29 agent-designed concepts all failed** — the rules below encode why, so you don't pay for
those failures again.

**Orchestration:** use the Workflow tool for the multi-agent phases (this skill's instructions
are your authorization to call it). If Workflow is unavailable, fan out with parallel Agent
calls instead. Do the refinement and production phases yourself, in the main loop — they need
taste and a tight visual feedback loop, not parallelism.

**Depth is adaptive.** Default: ONE concept round + critics + judges, then verification
(~1.5M agent tokens). Escalate to another concept round only while the judges return
"not shippable" — and before each new round, distil what killed the previous one into laws
(see below). If the user says "thorough", "no budget limit", or similar, run 2–3 rounds
regardless. If they say "quick" or "simple", run one round and skip the final verification
panel, but never skip the critics.

## Phase 0 — Research (do this yourself, ~15 min)

1. **Fetch every reference the user gave.** Website: WebFetch for copy/positioning AND, if a
   browser tool is available, execute JS on the live page to extract real computed tokens —
   colors, fonts, existing logo SVGs. Record verbatim headline copy; the brand's voice lives there.
2. **Mine the name.** The single highest-leverage question in the whole process: *is there a
   mark hiding inside the name itself?* (FedEx arrow, the `ai` inside cogn**ai**tiv.) A
   name-derived mark is unstealable and self-explanatory; free abstraction is where the 29
   failures came from. Write down every substring, ligature, letterform and double-reading
   the name offers.
3. **Scan the competitive field** with WebSearch: direct competitors, name-adjacent companies,
   and — critically — **who owns marks built on the same letters**. A monogram letter that a
   name-adjacent competitor already owns is a trademark trap, not an option (see laws.md, Law 9).
4. **Write `brand/BRIEF.md`** in the working directory: company, positioning, verbatim voice,
   what the mark must say (one sentence with a subject, verb and object), existing tokens,
   the name-asset analysis, competitor/trademark findings, the quality bar, and the
   deliverable list. Every agent you spawn reads this file first — write it for them.

## Phase 1 — Strategy (yourself + one palette agent)

- **The message sentence.** Force the brand's meaning into one sentence of the form
  "we [verb] [noun] [preposition] [noun]" before anyone draws. A mark can carry one
  relationship, not three adjectives.
- **Palette.** Spawn one agent to propose 3 complete palette directions with exact hex,
  WCAG contrast ratios on both dark and white grounds, precedent brands, and risks.
  Constraints that are non-negotiable: works on near-black AND white; survives one-colour
  reproduction; **no gradient as structure** (a gradient is a skin — it averages to mud at
  16px and cannot be embroidered). Treat any "this colour position is unclaimed" claim as
  unverified until you have checked the actual competitor set — this exact claim was made
  and was wrong in the source engagement. Pick one direction; a single confident flat accent
  beats any gradient.
- **Set up the render harness now** (scripts/ in this skill — copy them into the project's
  `build/` directory and adapt paths). Verify headless Chrome renders an SVG before any
  design agent needs it. Nothing else in this process works without the visual loop.

## Phase 2 — Concept rounds (Workflow)

Read `references/agent-prompts.md` for the full prompt patterns and structured-output
schemas. The shape of a round:

1. **4–6 designer agents in parallel**, each locked to a distinct strategic territory
   (derive territories from the brief; always include at least one name-derived territory —
   in the source engagement it was the only one that survived). Every designer must:
   - write 10+ one-sentence **phone tests** *before* touching coordinates (idea-first —
     craft applied to a weak idea produces a well-made bad logo);
   - construct on a 64×64 grid with a 4-unit module (see laws.md for the numeric floors);
   - **render their own mark and LOOK at it** with the preview script, minimum three
     draw→render→look cycles, and report what they *saw* at 16px, not what they intended.
     This is the single most important instruction in the skill: across two blind rounds,
     zero agents rendered their work and every one shipped a drawing that contradicted its
     own description.
2. **One devil's-advocate critic per concept** (pipeline, not barrier — critique each as it
   lands). Critics render the file themselves first, then audit against every law, check
   prior art with WebSearch (companies AND icon sets — a logo that is already a UI glyph is
   dead), name what the mark degrades into at 16px, list unintended readings exhaustively,
   and issue kill / rework / advance with a ceiling score.
3. **Two independent judges** (a design director weighting simplicity + distinctiveness, and
   a brand strategist judging "would a sceptical buyer trust this"). Judges must render every
   file themselves — never let them score from the dossier text. Shipping bar: **42/60**.
   Judges return exact, coordinate-level rework instructions, and ideas worth grafting
   from the losers.
4. **If nothing clears the bar:** write/extend `brand/LAW.md` with what killed this round —
   phrased as general kill-conditions, not complaints — and run the next round with the laws
   as mandatory reading. This is how the process converges instead of thrashing: round 3 in
   the source engagement succeeded *because* rounds 1–2 taught Laws 11–15.

## Phase 3 — Refinement (yourself, in the main loop)

Take the winning concept and the judges' instructions and finish it by hand. Build 3–5
geometry variants per open question, render contact sheets, choose with your eyes, measure
with the optics script (ink %, centroid, bbox, minimum clearances). Do not delegate this —
it is 20 minutes of taste that agents consistently fumble. Verify every judge instruction
against a render; judges are sometimes wrong in the details (in the source engagement the
judge's own "tested" fix had collapsed proportions no one would ship).

## Phase 4 — Verification (Workflow, skip only if user asked for "quick")

Three parallel adversarial verifiers, each with a distinct lens, each instructed to *refute*:
- **Legal/prior-art:** trademark collisions, letter-adjacent competitors, icon-set overlap,
  diacritic/foreign-character readings, "is this too generic to protect".
- **Production:** every deliverable rendered at true device sizes **in true colour** (make
  sure the harness does not force-override fills — the source engagement's harness hid the
  accent colour from every reviewer for three rounds), circular-mask survival, ink-gain,
  embroidery minimums, contrast arithmetic re-verified.
- **Unintended readings:** every size, both grounds, blurred, monochrome; letters, glyphs,
  anatomy, politics; and the "room test" — does the load-bearing claim survive a stranger.

Then one final judge scores against the 42/60 bar and splits findings into *must-fix before
delivery* vs *document as limitation*. Fix the must-fixes. Report the limitations honestly
in the README — a kit that hides its known weaknesses fails its owner later.

## Phase 5 — Production (yourself)

Read `references/deliverables.md` for the full manifest, the technical contract, and the
construction maths (lockup baseline rule, ink-radius rule for circular masks, self-theming
favicon pattern, wordmark outlining with fontTools). Build:

- Logomark: dark / light / mono-positive / mono-knockout / all-accent, tight artboard
- Wordmark: true font outlines (never live text), accent letters as their own path group
- Lockups: horizontal + stacked, mark's **foot on the wordmark's baseline**, size derived
  from a stated geometric relationship, not eyeballed
- Icons: square, app (plated), **round-safe avatar** and **adaptive foreground** scaled by
  *maximum ink radius* — a circular mask clips against ink radius, not the bounding box
- Favicon: ONE file with an embedded `prefers-color-scheme` style block
- `-themed` variants using `currentColor` + `var(--accent)` for inlining
- PNG rasters (favicons 16/32/48, apple-touch 180, adaptive 432, avatars, marks, lockups)
- `guidelines.html`: **generate it from the shipped SVGs** (inline them, draw the
  construction plate from the real geometry) so it can never drift from the assets. Publish
  as an Artifact when available.
- `README.md` with a pick-the-right-file table, the circular-mask warning, and the honest
  known-open-items list
- Package everything into `<company>-brand-kit/` (`svg/mark|wordmark|lockup|icon|themed`,
  `png/`, README, guidelines) and zip it.

## Working style

- Show the user real renders at every phase gate — contact sheets and in-situ mockups, not
  descriptions. Send files as they're produced.
- Report scores and failures honestly, including your own. "Round 1: all six concepts
  rejected, here is why" builds more trust than a highlight reel, and the user needs the
  failure reasons to steer.
- Between long agent runs, keep building the parts that don't depend on the outcome
  (wordmark pipeline, harnesses, guidelines generator).
- Track the laws file across rounds; it is the process's memory and the reason quality
  compounds instead of resetting.

## References

- `references/laws.md` — the design laws with their numeric floors and the banned-cliché
  list. Load into every designer/critic/judge prompt.
- `references/agent-prompts.md` — proven prompt patterns + JSON schemas for designers,
  critics, judges, verifiers, and the palette strategist.
- `references/deliverables.md` — asset manifest, SVG technical contract, lockup/ink-radius/
  favicon construction maths, packaging layout.
- `scripts/preview.mjs` — render-and-look harness (colour-true + mono rows, raster ladder,
  silhouette, circular crop, ink-gain). The core of the visual loop.
- `scripts/sheet.mjs` — multi-mark contact sheets (dark/light/knockout/ink columns).
- `scripts/context.mjs` — in-situ mockups (browser tab, app grid, avatar, cards, size ladders).
- `scripts/optics.py` — measure a rendered mark: ink %, centroid, bbox, max ink radius.
- `scripts/outline.py` — outline text from a TTF into SVG paths with real kerning (fontTools).
