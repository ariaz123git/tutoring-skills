---
name: add-diagrams
description: >
  Draw the setup diagram a person would sketch to solve a problem — a rough-but-literal
  labelled figure — and, when it materially helps, place it as the FIRST step of the worked
  solution before any equation. Covers situation sketches, free-body diagrams, motion diagrams, vector/component
  triangles, ray/circuit/geometry figures, and graph sketches. Static, computed-geometry SVG.
  Use when asked to "add a diagram / figure / sketch," "draw the problem," "show the setup,"
  "add free-body diagrams," or to make worked solutions open with a picture. Distinct from
  add-visuals (which builds INTERACTIVE animations/explorers). NOT for building the
  lesson/homework content itself (use create-lesson / create-homework).
allowed-tools: WebSearch WebFetch Read Write Edit Bash
---

# Add Diagrams

The first thing a good solver does is **draw the problem**: a rough but literal sketch with
every given labelled on it, the unknown marked, and a coordinate/sign convention chosen. That
one move turns a wall of words into a structure the math falls out of — and it is exactly the
step students skip. So the rule this skill enforces:

> **When a diagram materially helps solve the problem, open the worked solution with the
> diagram a person would actually draw — Step 1, before any equation. Skip it when it adds no
> reasoning value.**

**Why (research):** expert and textbook problem-solving strategies universally begin with
"construct a rough but literal picture and label all known information," then list
knowns/unknowns beside it (OpenStax *College Physics* §4.6; Pólya's *How to Solve It* — "draw
a figure, introduce suitable notation, separate the parts of the condition"). In physics
specifically, students who draw a **correct** free-body or motion diagram are significantly
more likely to solve the problem, and strong students use the diagram both to solve **and to
check** their answer (Rosengrant, Van Heuvelen & Etkina, *Phys. Rev. ST Phys. Educ. Res.*
2009; Van Heuvelen's multiple-representations / ISLE work). Multiple representations beat
equations-only. See *Sources*.

**Relationship to `add-visuals`:** `add-diagrams` draws the **static setup sketch** that
*starts* a solution (the thing you draw on paper first). `add-visuals` adds **interactive /
animated** exploration (play, scrub, sliders) when motion-over-time or what-if adds
understanding. A solution can have both: the diagram first, an optional animation later.

## Order of operations (one pass)

1. **Read the problem and build the mental image.** Decide the single diagram a solver would
   actually draw *first* (see *The catalog*). Usually one; occasionally a pair (e.g. a
   situation sketch + a free-body diagram).
2. **Draw the situation, literally but simply** — schematic, not art. Put **every given on the
   figure** (symbol + value + unit), **mark the unknown** with a `?`, show the **coordinate
   axes / positive direction**, draw **vectors as labelled arrows** and **angles as arcs** from
   the reference direction with their value.
3. **Compute all geometry in code** — vertices, arrow tips, arc endpoints, component legs.
   Never eyeball coordinates (helpers in `references/diagram-helpers.md`). Match the target
   file's palette using **literal hex** — not CSS `var()`: the figure must also render in a PDF
   export, and WeasyPrint's SVG renderer ignores `var()` (a var-coloured arrow renders black).
   **Compute label placement too** — see *Labels* below. Geometry is the half that logic checks
   can confirm; labels are the half that only looking can.
4. **When justified, place it as Step 1 of the solution**, before any equation, and have the
   working refer back to it ("from the diagram, the drop is Δy = −h…"). In homework, keep it
   inside the solution reveal or behind a later representation hint so it stays attempt-first.
5. **Verify** (see *Verification*): geometry lands correctly, labels match the givens and the
   final answer, and it renders cleanly at the target width.

## The catalog — which diagram would a person draw?

Pick the one that captures *this* problem's structure. If a diagram wouldn't help a solver,
don't add one.

- **Situation sketch** (the default) — draw the scenario (ball, ground, cliff, ramp, table,
  two objects on a road…), label every given with its symbol on the figure, mark the unknown,
  and show the axes / positive direction. For kinematics, projectiles, and most word problems.
- **Free-body diagram** — for force/dynamics problems: the object as a dot or box, **every
  force as a labelled arrow** pointing the way it acts, with axes. Prefer *not* to
  pre-decompose forces into components in the FBD itself — decompose in a separate step —
  which research finds works better (Rosengrant & Van Heuvelen).
- **Motion diagram** — dots at equal time intervals with velocity (and, where it matters,
  acceleration) vectors; instantly shows speeding up, slowing, or turning. (Van Heuvelen.)
- **Vector / component triangle** — a right triangle resolving a vector into its components
  with the angle arc; makes the sin/cos and Pythagoras step obvious. Use whenever a
  magnitude+direction must become components (or vice-versa).
- **Graph-as-diagram** — when the problem *is* about a graph (d–t / v–t / a–t, or any x–y):
  sketch the axes and the described shape, and mark the read-off (the slope, the area, the
  intercept) that answers it.
- **Ray diagram · circuit · geometry figure · bar model · number line** — for optics,
  circuits, geometry, and arithmetic-structure problems respectively (the "for any problem"
  cases — the diagram-first heuristic is general, not physics-only).

## What every problem-solving diagram must have

- The **situation**, drawn simply and schematically (a box for a car, a blob for a hill).
- **Every given labelled on the figure** — symbol = value (unit), placed where it belongs.
- **The unknown marked** — a `?` at the quantity being solved for.
- **A coordinate system / sign convention** — axes, or a small "+↑ / +→" marker — so every
  sign in the algebra is unambiguous.
- **Vectors as arrows** (length roughly to scale, labelled); **angles as arcs** from the
  reference axis with the value on the arc.
- A **one-line caption** naming what it shows.
- Nothing more. It is a *thinking tool*, not a finished illustration — clutter defeats it.

## Labels — where figures actually go wrong

Every recurring figure defect is a label defect, and none of them are caught by anything except
looking. Place labels by **computing the side**, never by a hand-tuned `{dx,dy}` — an offset
that is right for one geometry is wrong the moment an angle or a scale changes.

| Situation | Rule |
|---|---|
| Labelling a side/vector of a shape | Offset perpendicular from its midpoint, on the side **away from the shape's centroid** (`sideLabel(..., {away: centroid(...)})`). |
| Labelling an angle | A wedge is only `2·R·sin(span/2)` wide. Below **~25°** a numeric label will overprint an arm: label the arc **`θ`** and put `θ = 11.0°` in a clear corner (`angleLabelPlan`). |
| Any label near a stroke | Give text a **white halo** (`paint-order`) — a safety net for near-misses, not a substitute for placement, and WeasyPrint may ignore it. |
| Label near the canvas edge | Keep a **≥ 4 px margin**. `text-anchor="end"` at a small x silently runs off the viewBox. |
| Two panels in one figure | They need a real gutter — labels grow inward from each panel's far end and meet in the middle. If they collide, **stack the panels or emit two separate figures**; never shrink the type to fit. |

**Run `auditLabels` on the emitted SVG before rendering.** It approximates text width, so a
clean result means "worth looking at now," not "correct" — but it catches clipping and overlap
cheaply, and any warning is a defect rather than a suggestion.

## Implementation

- **Static inline SVG**, generated by a small author-time script (Node or Python) that
  **computes the geometry** and emits the SVG string to paste in — the same discipline as any
  computed figure (accurate arrows/arcs/triangles, not hand-placed). Reuse `diagram-helpers.md`.
- Use **literal palette hex** for colour (matching the host file), not CSS `var()` — the
  figure must render in the PDF export too, where `var()` silently renders black. Size for
  clarity (give angles and component triangles room); don't depend on a math font.
- Wrap it as the solution's first step, e.g.
  `<div class="dgm"><div class="dcap">Draw it — the setup</div>[svg]</div>` then the algebra
  follows and references it. Keep it print-friendly (it should show in a PDF export too).

## Verification (ship no wrong diagrams)

> Mechanics — headless capture, long pages, PDFs, low-resolution artifacts — are in
> **`../verify-material/references/rendering-and-proofing.md`**. Read it before the first render.

- **Geometry computed, not eyeballed** — arrow tips, triangle vertices, and arc endpoints land
  where the numbers say (an arrow labelled 40° actually points 40° from the axis).
- **Labels match** the problem's givens and the quantities in the final answer; the sign
  convention on the figure matches the one used in the algebra.
- **`auditLabels` clean** — no clipping, no overlapping text — *before* you spend a render.
- **Render to an image and look at it.** **Assume the first render is wrong.** Labels inside a
  shape, labels overprinting a line, text clipped at the edge, and panels colliding across a
  gutter are the normal first-pass result, not the exception — so budget
  *generate → audit → render → look → fix → re-render*. A figure that passed on the first look
  usually means you looked at a thumbnail.
- **Proof figures on their own page first** — all of them, one per row, at true size, before
  they go into the material. Defects are far easier to see without the surrounding prose.
- **Don't judge glyphs from a downscaled render** — `²` reads as `³` at small scale. Confirm
  against the source entities before "fixing" anything.

## Quality checklist (must pass all)

- [ ] Every diagram is instructionally justified; every worked solution that materially
      benefits from one **opens with it as Step 1**, before any equation.
- [ ] The diagram is the one a person would actually draw for *this* problem (right catalog type).
- [ ] Every **given is labelled on the figure**; the **unknown is marked**; **axes / sign
      convention** shown; vectors are labelled arrows and angles are arcs with values.
- [ ] Schematic and uncluttered — a thinking tool, not an illustration.
- [ ] **Geometry computed in code**; palette matches the host file; caption present.
- [ ] **Label placement computed** (side chosen away from the centroid; narrow angles use a
      symbol + legend); halo on text; `auditLabels` returns no warnings.
- [ ] The algebra **refers back** to the diagram (labels are reused, not reintroduced).
- [ ] Rendered to an image and eyeballed (no clipping/overlap; correct at target width) — and
      **re-rendered after the fixes that first look produced**.

## Anti-patterns

- A representation-heavy solution that jumps to equations without the diagram needed to make
  the setup intelligible.
- A pretty illustration that omits the givens, the unknown, or the sign convention.
- Eyeballed geometry (an arrow that says 40° but isn't; a triangle that doesn't close).
- **Hand-tuned label offsets** (`{dx:+12,dy:-8}`) instead of a computed side — right once, wrong
  after any change to the geometry, and the commonest source of labels inside the shape.
- **A numeric angle label crammed into a narrow wedge**, where it prints on top of its own arm.
- **Two panels squeezed into one figure** until their labels collide (stack them, or split the
  figure — shrinking the type is not the fix).
- Treating one render as the verification pass, or rendering only what happens to be above the
  fold on a long page.
- Cluttering the figure with every intermediate result — keep it the *setup*, not the whole solution.
- Drawing a diagram where one adds nothing (a trivial one-step plug-in).
- Decomposing forces inside the free-body diagram itself (do it in a following step).

## Sources

- OpenStax, *College Physics* §4.6 "Problem-Solving Strategies" (draw a rough but literal
  picture; label knowns; list knowns/unknowns). https://openstax.org/books/college-physics-2e/pages/4-6-problem-solving-strategies
- G. Pólya, *How to Solve It* (the "draw a figure" heuristic; introduce notation; separate the
  parts of the condition).
- D. Rosengrant, A. Van Heuvelen, E. Etkina, "Do students use and understand free-body
  diagrams?", *Phys. Rev. ST Phys. Educ. Res.* 5, 010108 (2009). https://link.aps.org/doi/10.1103/PhysRevSTPER.5.010108
- A. Van Heuvelen, multiple-representations / ISLE approach (sketch → motion & force diagrams →
  equations; qualitative representations before the math).
