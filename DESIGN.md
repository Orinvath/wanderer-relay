# DESIGN.md — how to design pleasing, readable interfaces
Researched for the Wanderer bench and all project UI. Sources: Nielsen
Norman Group (5 Principles of Visual Design; Gestalt series), UXmatters
(Visual-Design Principles and UX Best Practices), Toptal (Gestalt
framework), Elinext (composition series). For the Director and CC both.

## 1. VISUAL HIERARCHY — the eye must know where to go
- Arrange elements so attention lands in the intended ORDER: most
  important first. Achieved through SCALE, COLOR, CONTRAST, PLACEMENT,
  and SPACING — never through decoration.
- Rule: if everything is emphasized, nothing is. One dominant element
  per view; secondaries visibly quieter.
- Scale carries rank: larger = more important. Users read size as
  meaning before they read a single word.

## 2. VISUAL WEIGHT AND BALANCE — the seesaw
- Every element has WEIGHT: size, darkness, color saturation, density,
  and isolation all add weight. A small dark dense thing can outweigh
  a large pale one.
- BALANCE = equal distributed visual signal across an imaginary axis
  (usually vertical center). Symmetry is one way; asymmetric balance
  (a big light thing vs a small heavy thing) is livelier.
- Whitespace is a weight too — empty space balances filled space and
  is never wasted space. Crowding is the most common amateur fault.

## 3. CONTRAST — difference is meaning
- Contrast (color, value, size, shape) makes an element stand out and
  TELLS THE USER IT MATTERS. Use it only where meaning exists: a red
  element reads as danger/failure; do not spend red on decoration.
- Low contrast between text and background is a usability failure,
  not a style. Small text (under ~14px) needs the strongest contrast.

## 4. GESTALT — how the brain groups what it sees
- PROXIMITY: things near each other read as related. Group by spacing
  BEFORE adding boxes or lines — spacing is cleaner.
- SIMILARITY: same shape/color/size reads as same kind. Repeated
  iconography gives elements an identity.
- COMMON REGION: a shared background/border groups strongly — use
  sparingly (see proximity first).
- CONTINUITY: the eye follows lines and curves; flows should run one
  direction (our lanes law: data flows DOWN).
- CLOSURE: the mind completes incomplete shapes — outlines and partial
  frames are enough; heavy full boxes are rarely needed.
- FIGURE-GROUND: the subject must separate from the background
  instantly. If the user has to work out what is foreground, the
  layout failed.

## 5. COMPOSITION AND LAYOUT
- GRID: align to a grid; alignment is the cheapest form of order.
  Misalignment reads as error even when users cannot name it.
- RULE OF THIRDS: key elements at the intersections of a 3x3 grid
  make compositions dynamic; dead center is static (fine for status,
  dull for heroes).
- FOCAL POINT: one per view. Everything else supports it.
- READING PATTERNS: F-pattern for text-heavy views, Z-pattern for
  sparse ones — put what matters on the pattern's path.
- LINE LENGTH: shorten long text lines; balance via whitespace.

## 6. UX LAWS (the measured ones)
- FITTS: bigger and closer targets are faster to hit. Controls users
  press often must be big and near.
- HICK: more choices = slower decisions. Trim options; collapse the
  rare ones.
- MILLER / CHUNKING: working memory holds ~7 items; group anything
  longer (our one-question-at-a-time law is this).
- JAKOB: users expect your UI to work like the UIs they already know.
  Novelty in LOOK is fine; novelty in MECHANICS costs learning.
- AESTHETIC-USABILITY: attractive things are perceived as easier to
  use — polish is not vanity, it buys patience.

## 7. MOTION AND LIVE DISPLAYS (for the vitals and bench)
- Motion must MEAN something: every animation maps to a real event
  (our law: a flash is one processed thing; idle is dark).
- Continuous intensity beats blinking at high rates: aggregate events
  into glow strength; exact numbers ride beside.
- Failure state overrides activity state — red pins.
- Direction of motion = direction of data. One flow direction per
  view.

## 8. HOUSE RULES (Lonnie's laws, binding)
- Visual-spatial first: diagrams over text; structure over prose;
  logical chunks that read as blocks.
- NO fading or brightness tricks for meaning — COLOR ONLY, full
  presence (229).
- His artwork is never altered, only tinted/underlined (238 era).
- Legends on the page — a display explains itself (180).
- One dominant question/action per view; never a wall.
- Dark bench aesthetic: thin strokes, sigil-style marks, muted labels
  (letterspaced small caps), meaning carried by color and motion.
- Nothing "displays right" until HIS EYE passes it (118).

## 9. INFORMATION ARCHITECTURE (IA) — the structure IS the design
Sources: Toptal IA guide; freeCodeCamp IA/userflows; DFD readability
rules (Excalidraw); pipeline architecture literature.
- IA = the structural design of information so users can FIND and
  FOLLOW: organization schemes (group by one logic — for us,
  PROCESSING ORDER), labeling (plain names), navigation (the eye's
  path), search (finding one thing directly).
- ONE ORGANIZATION SCHEME PER VIEW. Mixing schemes (some spatial,
  some categorical) is what makes a map unreadable.
- THE TWO REQUIREMENTS of a good architecture map: hierarchical
  PLACEMENT (where a thing sits tells its rank/order) and a LEGEND.
- DFD READABILITY LAWS: never cross data flows · short labels ·
  unique names · one flow direction.
- PIPELINE PATTERN: when the real system is sequential (ours is — a
  moment travels the tick in order), draw it as ONE SPINE: a single
  main path, stages in processing order, side systems attached as
  short stubs to the stage they serve — never wired across the page.
- PATH EMPHASIS: the path currently being taken is highlighted;
  everything else stays quiet. The viewer follows ONE thing.
