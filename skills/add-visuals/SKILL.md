---
name: add-visuals
description: >
  Add visuals to an already-built lesson or homework (interactive HTML): animated physical
  scenes, graphs synced to a scene, phase-by-phase players, interactive parameter explorers,
  or static labelled diagrams (vectors, triangles, free-body, compass). It works in three
  moves — AUDIT which items genuinely benefit, PLAN how each visual should behave, then BUILD
  and verify — in one pass. It is context-aware: in homework, visuals are attempt-first
  (toggle-gated, only on worked/demo items); in a lesson, they can be inline teaching aids
  that show by default. Use when asked to "add visuals/animations/diagrams," "illustrate
  these problems," "make this interactive," "animate this," or to enhance an existing
  lesson/homework. NOT for building the lesson or homework itself (use create-lesson /
  create-homework) or a course map (use create-syllabus).
allowed-tools: Read Write Edit Bash
---

# Add Visuals

Take an existing lesson or homework and layer in visuals **where they add understanding a
sentence can't** — motion over time, the link between two representations, geometry and
direction, or how an answer changes with an input. A visual that just decorates a problem is
noise; the whole job is to place a *small number* of well-behaved visuals exactly where they
earn their place, and to implement them without the bug class that plagues hand-rolled
animation (see *Must-avoid bugs* — this is the point of the skill).

> Reusable, bug-safe code and the verification harness live in `references/patterns.md`.
> Read it before implementing anything animated.

## Order of operations (one pass: audit → plan → build → verify)

1. **Audit.** Read the target file. Detect whether it's a **lesson** (visuals can teach
   inline) or **homework** (visuals are attempt-first). List every candidate item — worked
   examples, key derivation steps, concepts, questions — and decide which genuinely benefit
   using *When a visual earns its place*. Be selective: usually a handful, not every item.
2. **Plan each visual.** For every item you'll enhance, pick a type from *The catalog* and
   write a short **behaviour spec** (below) *before coding* — what it depicts, which values
   update live, the control, the synced views, the reveal rule. Keep the spec in a comment or
   a scratch note so the behaviour is intentional, not emergent. Do this in the same pass —
   no separate approval gate — but do it as an explicit step, not on the fly.
3. **Build.** Implement with the patterns in `references/patterns.md`, **reusing the target
   file's existing palette / CSS variables** so the visuals look native. Apply the context
   rule (gated+lazy in homework; inline in a lesson).
4. **Verify** (see *Verification*). Any displayed number checked in code; the JS tested in a
   real DOM that simulates browser quirks; the visuals rendered to images and eyeballed.

## When a visual earns its place (audit criteria)

Add a visual only when it does something prose can't:

- **Process / motion over time** — something moves, accelerates, fills, or changes state.
- **Relationship between representations** — a scene ↔ its graph, or two linked graphs
  (e.g. position–time next to velocity–time), so the student sees how one implies the other.
- **Geometry / direction** — vectors, components, angles, bearings, free-body layouts.
- **Parameter sensitivity** — "what happens to the outcome as I change this input."

Do **not** add one to: pure recall items, trivial one-step problems, or — in homework — any
**cold / attempt-first item** (a visual there pre-empts the student's own attempt). One
visual per item at most; resist the urge to visualize everything.

## Context rule (lesson vs homework)

- **Homework → attempt-first.** Toggle-gated behind a "Show visualization" button, hidden
  until pressed, built lazily on first open, and placed **only on worked/demo items** — never
  on the cold sister/capstone/practice questions.
- **Lesson → teaching aid.** May render inline and animate/reveal as part of the explanation
  (a "Play" or "Slope/Area"-style control is fine); it's there to teach, not to be earned.

Detect context from the file (is it a homework with sister/attempt-first sections and staged
hints, or a lesson?). When ambiguous, default to attempt-first/gated.

## The catalog (visual types + when to use)

- **Animated physical scene** — an object moving along a track/scene with **live value labels
  placed next to what they measure** and state cues (e.g. brake lights while decelerating). For
  real-world process/motion.
- **Synced graph panel(s)** — one or more graphs (position–time, velocity–time,
  acceleration–time, or any x–y relationship) with a scrubber/Play, a moving marker, a drop
  line to the axes, and (for "area = …" ideas) a fill that grows. Pair a graph with the scene
  so the two update together. Include a **live readout** of the same quantities.
- **Phase-by-phase player** — for multi-stage processes: play one phase, **freeze that
  phase's numbers onto a card that stays on screen**, then unlock the next; show all cards
  plus a total at the end. Reveals the graphs/scene progressively as phases play.
- **Interactive parameter explorer** — a slider changes an input and every view updates live;
  for sensitivity/what-if.
- **Static labelled diagram** — vectors, right-triangles (for component ↔ magnitude/angle),
  free-body diagrams, compass/axis keys. Size for clarity, label angles with an **arc** from
  the reference axis, caption it in one sentence, and **compute the geometry in code** (arc
  endpoints, vertices) rather than eyeballing coordinates.

## Behaviour spec (define before coding)

For each visual, decide and write down:

- **Depicts:** the scenario/quantity being shown.
- **Live values:** which numbers update as it plays/scrubs, **labelled in-scene next to the
  quantity they measure** (on the radius, the vector, the angle arc — labelled-diagram style),
  not only in a separate readout. See `patterns.md` §7a.
- **Solution diagram:** each worked/demo item's solution shows a human-style labelled setup
  sketch (givens tagged in-scene, accurate geometry) — `patterns.md` §7b.
- **Control:** scrub slider · Play/Pause · phase buttons · parameter slider (pick one model).
- **Synced views:** which panels move together (scene + graph(s)).
- **Reveal rule:** gated+lazy (homework) or inline (lesson).
- **Data source:** the exact formula/values driving it — verified in code first.

## Must-avoid bugs (hard rules — this is why the skill exists)

Hand-rolled animation reliably ships this exact set of bugs. The patterns in
`references/patterns.md` encode the fixes; do not deviate:

1. **Never use an input widget's value as the animation clock.** A range `<input>` **snaps
   its value to its `step`**, which silently (a) breaks "restart from the end" (the stored
   value is a hair below max) and (b) makes motion stutter (each frame re-snaps and loses the
   sub-step remainder). Keep an **internal floating-point clock**; write to the slider for
   display only, never read it back for state.
2. **One uniform time-rate across segments/phases.** Don't clamp each segment's duration
   independently, or short segments crawl and long ones race. Compute a single
   ms-per-unit-time and make each segment proportional to its span.
3. **Multi-panel SVGs need a flex wrapper with capped widths and a `max-height`.** Sibling
   panels without a flex container are full-width blocks, so a near-square SVG balloons to
   full-width-tall — unrenderable and laggy from the huge repaint. Wrap panels, cap widths,
   cap `max-height`.
4. **Guard the animation loop.** Cancel any running frame before starting; guard against
   double-start; reset state cleanly on replay so restart always works. **In a phased player,
   a control disabled while a stage animates must be re-enabled when that stage finishes** —
   re-enable on *every* non-final stage, not just the last, or only stage 1 is ever playable.
5. **Lazy-build on first open** (for gated visuals) — keep the initial DOM clean and cheap.
6. **Render inline math at normal body size** — oversizing misaligns radicals and inflates
   scripts. Match the file's existing math styling.

## Verification (ship no bugs)

> Render mechanics — headless capture, pages taller than the viewport, forcing collapsed panels
> open, and the render artifacts that fake a defect — are in
> **`../verify-material/references/rendering-and-proofing.md`**. Label-placement rules (compute
> the side, narrow-angle arcs, halos, the pre-render `auditLabels` check) are in
> **`../add-diagrams/references/diagram-helpers.md`**; they apply to any labelled SVG, animated
> or not.

1. **Verify any displayed numbers in code first** (with domain cross-checks, not just
   arithmetic) — a visual that shows a wrong value is worse than no visual.
2. **Test the JS in a real DOM (e.g. jsdom) and simulate the platform's quirks.** Logic-only
   reasoning and *naive* DOM tests pass code the real browser breaks — you **must** simulate
   **range-input step-snapping** (patch the input `value` setter to snap to `step`) or you'll
   miss the restart/stutter bug. Drive the animation frame-by-frame with a mocked clock; probe
   a **full-precision** attribute (not a rounded readout); assert: strictly-advancing motion
   with no stall frames, uniform per-frame steps, correct restart from the end, each phase
   completes and stores its card. (Harness in `references/patterns.md`.)
3. **Render the visuals to images and look at them.** Many defects are visual only — clipped
   axis labels, colliding tick/axis text, panels too tall, misaligned math. Render (headless
   capture, or WeasyPrint for static SVG) and eyeball before shipping. **Also confirm each curve
   meets its axes at the expected values** — a point at the axis value must land exactly on the
   axis line (a curve starting above/below it means the data's value→pixel range and the drawn
   axis disagree). Note this is only a backstop: a render built from the same buggy mapping hides
   the defect, so rely on the shared-mapping rule in `patterns.md`, not just the eyeball.
4. **Tooling gotcha:** the file you edit is the source of truth; a shell/cache view can lag
   and show a stale/truncated copy. Verify against the authoritative file layer; when testing
   in the shell, write a fresh copy the shell owns rather than reading back through a mount.

## Quality checklist (must pass all)

- [ ] Visuals added **only where they add understanding prose can't**; not every item; one per item.
- [ ] **Context rule** applied — gated + lazy + demo-items-only in homework; inline teaching aids allowed in a lesson.
- [ ] Each visual has a written **behaviour spec** (depicts / live values / control / synced views / reveal / data source).
- [ ] Reuses the target file's **palette/CSS variables**; looks native.
- [ ] **Internal float clock** drives animation (never the slider value); **uniform** rate across phases; loop guarded; restart works.
- [ ] Multi-panel SVGs are **flex-wrapped, width-capped, `max-height`-capped**; nothing balloons.
- [ ] **Graph axes/gridlines drawn from the same mapping as the data** — a value at the axis lands *on* the axis (not above/below/overshooting).
- [ ] Live **readouts** show the same values; math at normal size.
- [ ] Live values are **labelled in-scene next to what they measure** (labelled-diagram style), not only in a readout box.
- [ ] Worked/demo **solution reveals include a human-style labelled setup diagram** (accurate geometry, givens tagged in-scene).
- [ ] Numbers verified in code; **JS tested in a real DOM with range-snapping simulated**; visuals **rendered to images and eyeballed**.

## Anti-patterns

- Decorative visuals that don't add understanding; visualizing every item.
- Values shown only in a readout box instead of **labelled on the thing they measure**; a worked solution with no setup diagram.
- Putting a visual on a cold/attempt-first homework question (pre-empts the attempt).
- Driving the animation off the range input's value (breaks restart, causes stutter).
- Per-segment clamped durations (uneven perceived speed).
- Sibling SVG panels without a flex wrapper / size caps (too-tall, laggy).
- Rebuilding a large readout's `innerHTML` every frame when a cheap `textContent`/attribute
  update would do.
- Shipping an animation "verified" only by logic/naive jsdom (missing the step-snapping bug),
  or a visual never rendered and eyeballed.
- Oversized inline math that misaligns radicals and superscripts.
