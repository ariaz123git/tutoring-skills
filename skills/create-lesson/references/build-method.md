# Build Method — full patterns for golden lessons & homework

Reference detail for `create-lesson` (and shared by `create-homework`). The SKILL.md
gives the rules; this file gives the *how*, distilled from the build process that
produced the best materials (the projectile-motion lesson).

## 1. Fading worked examples → independent practice (three nested fades)

Support is removed gradually at three scales so the student always attempts first and
pulls only the smallest hint needed:

- **Within a part** — use the smallest useful adaptive hint sequence: no hint for direct
  retrieval, one strategic clue for a single bottleneck, or several linked informational /
  Socratic / targeted-feedback prompts for genuinely distinct decisions. Do not force a
  nudge → method → math sequence. For homework, follow `../../create-homework/references/hint-design.md`.
- **Within a problem** — multi-part questions are more guided in early parts, less in
  later parts (the scaffold fades part-to-part).
- **Across the assignment** — guided "teaching" problems first (full staged hints), then
  unguided **"sister" problems** (a plain worked solution only, no staged hints) that
  mirror the taught ones so the student proves they can do it cold.

In the lesson, the rhythm is **Ask → Solve → Watch** per concept, and the bottom
**assigned practice uses adaptive part-level hints**; the unguided "sister" problems
(plain solution, no staging) are mainly the **homework's** cold check (see create-homework).

## 2. Visuals & animations

- **Tech:** HTML5 `<canvas>` + vanilla JS. Use **analytic (closed-form) physics**, not
  step integration — compute positions/velocities directly from the equations so the
  picture is exact and matches the math.
- **Two patterns:**
  1. **Concept demo** — a single idea made undeniable (e.g. drop-vs-shoot: two objects,
     a dashed line proving they fall at equal heights / land together).
  2. **Interactive lab** — sliders for the inputs, a live trajectory/curve, live numeric
     readouts (e.g. vₓ, v_y, height), and **preset buttons that load each worked
     example's exact numbers** so the student can watch the very problem they just solved.
- **Label values in-scene, on the thing they measure** (default): the radius carries its
  `r = …`, a velocity arrow is tagged `v = …` at its tip, an angle shows its value on its arc —
  live-updated, labelled-diagram style — not only in a separate readout box. (`add-visuals`
  `patterns.md` §7a.)
- **Worked examples include a human-style labelled setup diagram when the representation helps
  solve the problem** — the sketch, graph, sign chart, number line, or component view a tutor
  would actually use. Skip decorative diagrams for direct recall or routine algebra.
- **Curved graphs:** sample the curve in Python and embed as SVG (don't approximate by
  hand). Straight-line/relationship graphs can be drawn directly.
- Always include a `prefers-reduced-motion` fallback (static end-state).
- **If it animates, avoid the hand-rolled-animation bug class** (these ship silently):
  never use a range input's value as the clock — it snaps to its `step`, which breaks
  "restart from the end" and makes motion stutter; keep an **internal float clock** and
  write to the control for display only. Use **one uniform time-rate** across phases;
  **flex-wrap and size-cap** multi-panel SVGs; **guard/reset the loop** so replay works
  (and in a phased player re-enable the control on *every* non-final stage, not just the
  last). **Draw graph axes/gridlines from the same mapping as the data** (`axis = mapY(0)`,
  gridline for *k* at `mapY(k)`) so a value at the axis lands *on* it. For anything past a
  simple hook, use the **`add-visuals`** skill — it owns this depth.

## 3. Verification workflow (non-negotiable)

**Before any HTML**, compute every number in Python with `assert`s, including physics
cross-checks, e.g.:
- kinetic energy cannot increase in a collision;
- an explosion's centre-of-mass velocity = mass-weighted average of the fragments;
- elastic collisions reverse relative speed;
- inverse-projectile launch angles, back-substituted into the trajectory equation, hit
  the target.

**After building**, run structural validation:
- `node --check` the extracted JS (syntax);
- assert variable-length hint sequences reveal correctly, dependent hints unlock in order,
  cold/test mode exposes no coaching, and each hint advances a distinct decision;
- tag-balance / closing-tag check on the HTML;
- confirm any simulation's `calc()` reproduces the worked answers exactly;
- **sanity-read each worked solution for *method*** — Python asserts catch a wrong number, not a wrong approach;
- **render every visual to an image and eyeball it** — clipped labels, colliding text, a
  panel too tall, a curve that misses its axis; logic checks can't see these. **Assume the
  first render is wrong** and budget the fix-and-re-render; capture a long page in segments
  and force `<details>` open in a scratch copy, or the collapsed solutions are never seen;
  proof figures on a page of their own before they go into the material. Mechanics:
  `../../verify-material/references/rendering-and-proofing.md`;
- **place figure labels by computed side, not hand-tuned offsets**, and run the pre-render
  `auditLabels` check (`../../add-diagrams/references/diagram-helpers.md`) — labels landing
  inside a shape or off the canvas edge are the commonest figure defect;
- **when a check fails, verify the check before editing the file** — a hardcoded count or a
  mis-scoped substring test is about as likely to be wrong as the artifact; and never "fix"
  the source from a downscaled render (`²` reads as `³`);
- **test animated JS in a real DOM (jsdom) that simulates range-input step-snapping** (patch
  the value setter to snap to `step`), driving frames with a mocked clock — a naive DOM test
  passes the very restart/stutter bug the browser hits;
- **source-of-truth:** the file you edit is authoritative; a shell/mount view can lag and show
  a stale or truncated copy — verify against the edit layer, and when testing in the shell
  write a fresh copy it owns rather than reading back through the mount.

## 4. Scaffolding & voice

- **Symbol key + derivations:** give a short symbol key; where a formula is *derived*,
  show the derivation and cite the principle it comes from (don't just assert formulas).
- **Teach untaught techniques inline:** if a problem needs a tool the student hasn't met
  (e.g. the quadratic formula in a cliff projectile), teach it briefly in place rather
  than assuming it.
- **No editorial leaks:** student-facing voice only. No "he probably knows this,"
  "if shaky teach," "advance when," or other tutor coaching — that lives in the plan.
- **Notation discipline:** consistent subscripts (`<sub>`), units on every quantity,
  sign conventions stated once and kept.

## 5. One-line build checklist

Scope → backbone+sources → spine (big idea / misconceptions / objectives) → verify
numbers in Python → build HTML (hook+visual → checks → method → Ask/Solve/Watch ramp →
misconceptions → closure → assigned staged-hint practice) → post-build validation →
strip tutor voice.
