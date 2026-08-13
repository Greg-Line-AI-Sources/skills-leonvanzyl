# The Design Laws

Derived from a real engagement in which 18 concepts across three rounds were rejected by
adversarial review before one survived. Each law is a **kill condition** for concept review —
cite the law number in critiques. Where a law has a numeric floor, the number is load-bearing:
it comes from device-pixel arithmetic, not taste.

All coordinates assume the standard construction frame: a **64×64 viewBox** with a **4-unit
module** (at 16px favicon size, 4 units = 1 device pixel).

---

## LAW 1 — The message must be in the geometry, bilateral, and the parties must differ in kind
If the brief's sentence is "we [verb] X into Y", the mark needs two parties and a verb. A
single closed form with a passenger inside it says nothing. Two congruent halves can only say
"two of the same thing". The parties must be opposite in register: curved vs faceted, grown vs
machined, host vs guest. Most logo rationales are post-hoc fiction — critics must ask whether
the sentence is *drawn* or merely *told*.

## LAW 2 — The brand's key asset is present matter, never absence
If the brand's story is "X lives inside us", X must be bright, solid, and there. A hole that
the rationale calls X reads as an empty socket — the opposite claim. (Source engagement: two
concepts drew the AI as a void; both killed.)

## LAW 3 — Nothing may read as captive
Enterprise buyers' loudest fear is lock-in. Seated, not swallowed; structural, not trapped.
A mark whose thesis is "inserted and cannot be removed" is a threat rendered as a logo.

## LAW 4 — Negative space should vent to the silhouette
An enclosed counter fills in at small sizes and dies in embroidery/engraving/stamping. Prefer
voids open to the outline. **A deliberate exception is allowed for letterform counters that
measure ≥ 5 device pixels at 16px (≥ 20 units)** — verified by dilation testing in the source
engagement — but the exception must be stated and signed off, never silent, and a vented
small-size variant should ship for sub-15mm physical reproduction.

## LAW 5 — 6-unit minimum clearance; 8-unit minimum stroke
At 16px, a 4u gap is one device pixel and greys out; 3u vanishes. No gap below 6u anywhere.
No stroke/band below 8u (12u is comfortable). State measured minimums in every submission —
and measure them; designers reliably misreport their own clearances.

## LAW 6 — Straight seam edges on the 4-unit module
Every seam-defining straight edge on a multiple of 4 so it lands on whole device pixels at
16px. Only arcs carry non-integers. An off-module edge renders as a permanent 50%-grey
antialiased column. Minimise curve count; every arc's centre and radius must be derivable
from a small construction family, nothing placed by eye.

## LAW 7 — Name the 16px glyph before you defend the mark
Every mark degrades into *something* at favicon size. Name it before drawing, verify it after
rendering. The source engagement's failures degraded into: Pac-Man, the letter G, ∅, a pause
button, a loading spinner, Π, a damaged D, a SIM card, an L, a bar chart. If you cannot name
yours, you have not designed it.

## LAW 8 — No container crutch, no orb, and the outline must be ownable
A rounded-square container becomes the whole silhouette at 16px. A gradient disc is the
2020s AI-orb genre. Cover the inside of your mark: if the outline alone is a stock shape
(a font glyph, a primitive), the mark is not ownable — and single-letter marks with generic
outlines get registrations too narrow to enforce.

## LAW 9 — Check letter-adjacent competitors before committing to a monogram letter
Before any letterform mark: WebSearch for competitors whose *names are adjacent* to the
client's AND whose marks are built on the same letter. Name adjacency + mark adjacency +
overlapping services is the textbook trademark-opposition pattern. (Source: "Cognaitiv"
could not use a C — Cognite, Cognizant and Cognition AI all own bold C marks in adjacent
software.) This is a legal exposure question, not a taste question; it belongs in the brief
before drawing time is spent.

## LAW 10 — The strip test
One ink, on cotton, faxed, engraved. What survives? If the survivor is a letter of the
alphabet that is not the brand's own, a UI glyph, or punctuation — kill.

## LAW 11 — Build from the name before building from abstraction
Abstract relationship-objects (the hardest category in identity design) strip-test into
someone else's letter or a UI glyph with grim reliability — that failure mode killed 12 of
the source engagement's 18 concepts. The marks that genuinely solve "two things in relation"
(FedEx, Beats, Carrefour) are almost all name-derived. Mine the name first: substrings,
ligatures, double readings, a letter pair sharing a stroke. Always give at least one concept
round a name-derived territory.

## LAW 12 — Amends Law 10: the brand's own letters are a win
Strip-testing to the brand's OWN initials or name-fragment is a monogram, not a failure.
Any other letter, punctuation, or UI glyph remains a kill. But the claimed letters must
actually be in the drawing — see Law 16.

## LAW 13 — The two bodies must touch
Clearance on every face is a part that has not been installed; a logo cannot depict a verb
with air. Contact on one stated register face, relief elsewhere. The element carrying the
brand's core meaning must not be the one element floating free.

## LAW 14 — Render your work and look at it (the most important law)
Across two rounds, zero designers rendered their own marks; all shipped drawings that
contradicted their descriptions — arcs they declared absent, ink splits misreported by
3%+, glyph readings that were fantasy. Draw → render → **Read the image** → fix → repeat,
minimum three cycles. Critics and judges render files themselves and never trust dossier
text. A mark you have not seen at 16px is a mark you have not designed. Ensure the harness
shows TRUE COLOUR as well as forced mono — a harness that force-overrides fills can hide an
entire accent element from every reviewer (this happened; it survived three rounds unseen).

## LAW 15 — Name the 16px glyph out loud, from the render
After rendering: write down what a stranger calls the 16px cell. Check it against Law 12.
Then check icon libraries (Lucide, Material Symbols, SF Symbols, Font Awesome) — a logo that
is already a toolbar icon is dead in exactly the contexts the client lives in.

## LAW 16 — The claimed letters must survive the delete test
If the mark claims to be two letters, deleting the second letter's distinguishing element
must NOT leave the first letter complete and undamaged. An element that owns only its own
diacritic is a diacritic, not a letter. Each claimed letter needs exposed edge and a real
contribution to the outline. (Source: the final mark's "i" was entirely absorbed inside the
"a"'s band — delete the dot and a stock geometric "a" remained. Caught only in final
verification; check it at concept stage.)

## LAW 17 — Circular masks clip against ink radius, not the bounding box
Compute the maximum distance from bbox centre to any point of ink. Avatars need all ink
inside r=27 of a 64-frame's r=32; Android adaptive icons inside r=20 (the ~66dp guaranteed
circle). Square-container icons are never round-safe by accident. Ship dedicated round-safe
variants or the platform will render the mark amputated — or as a plain ring.

## LAW 18 — Accent discipline
One accent colour, flat, on ONE element — the element carrying the brand's core meaning.
Above ~10% of total ink an accent becomes the subject. A small accent dot in the top-right
of a square tile is the OS notification-badge position at icon sizes — check any corner-
placed accent against a control image with a real badge. Accent hex must clear WCAG 4.5:1
on at least one canonical ground, with a designated darker twin for the other.

---

## The banned list (cumulative, from real kills and prior-art collisions)

brains · circuits · robots · chat bubbles · node-and-edge graphs · hexagons · infinity loops ·
swooshes · spirals · orbs · atoms · neural-layer diagrams · gears/cogs · light bulbs · rockets ·
isometric cubes · letters in rounded squares · four-point sparkles · DNA · fingerprints · mazes ·
keyholes · puzzle pieces · apertures/irises (= loading spinner) · dovetail joints · USB/plug
pictograms · login/logout glyphs · leaves · yin-yang · Pac-Man silhouettes · weaves · shields ·
mountain peaks · abstract handshakes · anything already shipped by a major icon set

A concept whose first instinct matches this list gets discarded before construction. The list
is genre wallpaper: the buyer has seen each of these a thousand times and reads them as
template work.

---

## Process rules that ride with the laws

- **Phone test before coordinates.** One sentence a stranger could redraw the mark from.
  If the idea fails as a sentence, geometry cannot save it.
- **Ceiling scores.** Critics estimate the best score a concept could reach after perfect
  rework. Judges pick the highest ceiling, not the highest current score.
- **Grafts.** Every judge round lists specific ideas worth stealing from the losers.
  The winning mark in the source engagement carried techniques from four dead concepts.
- **Laws compound.** After a failed round, write what killed it as new numbered laws and
  make them mandatory reading for the next round. Round 3 succeeded because rounds 1–2
  became Laws 11–15.
