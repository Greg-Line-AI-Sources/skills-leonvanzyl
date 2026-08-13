# Agent Prompt Patterns & Schemas

Proven across three rounds of the source engagement. Adapt wording to the brand; keep the
*structure* — especially the honesty fields (`whatIActuallySawAt16px`, `renderMatchedDescription`),
which exist because agents reliably describe marks they did not draw.

Every agent prompt begins with mandatory reading:
```
Read, in this order, before doing anything:
1. <workdir>/brand/BRIEF.md   — company, message, tokens, quality bar
2. <workdir>/brand/LAW.md     — kill conditions (this file grows between rounds)
```

## 1. Designer agent (one per territory, 4–6 in parallel)

Territory design: each designer gets ONE strategic territory with a `lens` paragraph
describing the idea-space and its specific DANGERS (what nearby cliché it degrades into).
Always include a name-derived territory (Law 11). Territories from the source engagement
that produced the winner: "the two letters of the name fused so they share one stroke".

Prompt skeleton:
```
You are a senior identity designer at a AAA studio. [If a prior round failed: N concepts
have been rejected; you have the post-mortem in LAW.md. Do not repeat it.]

YOUR TERRITORY — <title>
<lens, including DANGERS>

PROCESS — in this order; the order is the point:
1. Write 10 one-sentence PHONE TESTS inside your territory. Ideas only, no coordinates.
2. Cull against the laws. For each survivor, NAME what it becomes at 16px (Law 7).
3. Pick one. Say why it beats the other nine.
4. Only now construct: 64x64 grid, 4u module, real coordinates, derived edges.
5. Write the SVG file to <concepts-dir>/<slug>.svg.
6. RENDER AND LOOK (Law 14): node <build>/preview.mjs <file>; Read the PNG. Look at the
   16px and 12px cells, the silhouette, the ink-gain row. Write down what you SEE.
7. Fix what you saw. Re-render. Minimum three cycles. Most of your effort goes here.
8. Self-attack for one paragraph; strongest objection goes in knownWeakness.

Colour: flat <bone/ink> with at most ONE element in <accent>, and that element must be
<the brand's core-meaning element>. No gradients.
```

Schema (StructuredOutput):
```json
{ "slug": "", "name": "", "oneLiner": "", "phoneTest": "",
  "whatIActuallySawAt16px": "reported from the RENDER; a description of intent voids the submission",
  "renderCycles": 0, "smallestGapUnits": 0, "curveCount": 0,
  "construction": "geometry precise enough for a stranger to rebuild",
  "whyItBeatsTheOthers": "", "knownWeakness": "" }
```

## 2. Devil's-advocate critic (one per concept, pipelined behind its designer)

```
You are a DEVIL'S ADVOCATE brand critic. [Prior rounds' critics correctly killed N concepts;
match that standard.] Be fair only in this sense: if a concept genuinely clears the bar,
say so — a blanket rejection with no path forward is itself a process failure.

FIRST ACTION, before any analysis: render the file yourself
(node <build>/preview.mjs <file> <out.png>) and Read the PNG. Judge what is THERE, not what
the designer says. Report any discrepancy as a credibility failure.

Then: 1) LAW AUDIT — every law, pass/fail, with coordinates/render evidence quoted.
2) PRIOR ART — WebSearch companies (especially name-adjacent, Law 9) AND icon sets.
3) 16px READING from the render — name the glyph a stranger would call it (Laws 12/15).
4) UNINTENDED READINGS — exhaustive: letters, glyphs, anatomy, politics, foreign scripts.
5) MESSAGE HONESTY — is the brief's sentence in the geometry or bolted on after (Law 1)?
6) STRIP TEST (Law 10).
Verdict: kill / rework / advance, the single highest-leverage fix, and a ceiling score /60.
```

Schema:
```json
{ "slug": "", "renderMatchedDescription": false, "lawFailures": [""],
  "priorArtCollisions": [""], "actual16pxGlyphName": "", "unintendedReadings": [""],
  "messageIsGenuine": false, "stripTestSurvivor": "", 
  "verdict": "kill|rework|advance", "highestLeverageFix": "", "ceilingIfFixed": 0 }
```

## 3. Judges (two, parallel, after all critiques land)

Judge A — **design director**: weights simplicity and distinctiveness hardest; "will this
still be right in 10 years and recognisable at 16px?" Judge B — **brand strategist**: does
not care if it is pretty; "would the brand's actual sceptical buyer trust the company behind
it?"; weighs trademark risk as business risk.

Both judges MUST render every concept themselves before scoring — give them the exact
command and the slug list. Score 0–10 on distinctiveness, simplicity, messageFit,
scalability, timelessness, craft (total /60; shipping bar 42). Below the bar, they still
name the highest-ceiling concept and return **exact, ordered, buildable rework
instructions** — coordinates and elements, no ambiguity, because the next stage implements
them literally. Plus grafts from the losers and `isAnyOfThisShippable`.

```json
{ "scores": [{ "slug": "", "distinctiveness": 0, "simplicity": 0, "messageFit": 0,
  "scalability": 0, "timelessness": 0, "craft": 0, "total": 0, "note": "" }],
  "ranking": [""], "winner": "", "winnerRationale": "",
  "exactReworkInstructions": [""], "graftFromRunnersUp": [""],
  "isAnyOfThisShippable": false }
```

## 4. Palette strategist (one, during Phase 1)

Propose THREE complete directions: name, positioning rationale, ink/surface/paper/primary
(+ darker twin) hex, WCAG ratios for the accent on both grounds (computed, and both must be
stated), what it signals, what it risks, precedent brands. Constraints: works on near-black
AND white; survives one-ink reproduction; no gradients. Require the agent to check
"unclaimed position" claims against the client's REAL competitor set — and re-verify the
winning claim yourself with WebSearch before adopting it (this claim was confidently wrong
once already). Rank and recommend one.

## 5. Final verifiers (three, parallel, Phase 4)

Framing for all three: "Your default assumption is that this mark has a fatal flaw and your
job is to find it. If after genuine effort you cannot land a fatal blow, say so plainly —
a false alarm is as damaging as a miss." Findings ranked FATAL / SERIOUS / NOTED, each with
rendered or measured evidence, plus `couldNotRefute` (what genuinely survived) and a ship
recommendation.

- **Legal/prior-art:** WebSearch aggressively; letter-adjacent competitors; icon sets;
  diacritic/foreign-script readings; "too generic to protect" (the strongest attack on any
  letterform mark — make it properly).
- **Production:** render every deliverable at true device sizes IN TRUE COLOUR (verify the
  harness does not override fills); circular-mask arithmetic (Law 17); ink-gain/embroidery;
  re-verify every stated contrast ratio; SVG source audit (fill-rules, viewBox, ids).
- **Unintended readings:** every size/ground/blur/mono; the room test — does the identity's
  load-bearing claim survive a stranger who hasn't read the rationale (Law 16: run the
  delete test).

Then one final director scores /60 against the bar and splits findings into
`mustFixBeforeDelivery` vs `documentAsLimitation`. Instruction that matters: "Do not invent
new objections to appear rigorous. If it is good, say it is good."

## Workflow-script notes

- Pipeline designer→critic per territory (no barrier); barrier only before the judges,
  who need the full field.
- Prefix every SVG id with the concept slug — multiple concepts get inlined into one
  contact-sheet page, and duplicate ids silently corrupt each other's gradients.
- Give every agent `effort: 'high'` for design/critique/judging; it is where the money is.
- Verify arc sweep flags explicitly in prompts: a wrong sweep flag draws a shape the
  designer never intended, and it happened repeatedly.
