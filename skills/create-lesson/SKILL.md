---
name: create-lesson
description: >
  Build a single-topic, student-facing teaching lesson (interactive HTML) that a tutor
  and student work through together, then assign the student practice to finish alone.
  Use when asked to create a "lesson," "teaching session," or guided walkthrough of a
  concept (especially math/physics). Produces a ramped, misconception-aware,
  reveal-by-step lesson with an animated visual hook, fully verified worked examples,
  and a bottom "try on your own" practice block with staged (one-step-at-a-time) hints.
  NOT for plans, standalone worksheets, or quizzes (use create-homework for assigned
  take-home work).
allowed-tools: WebSearch WebFetch Read Write Edit Bash
---

# Create Lesson

A lesson is "golden" when a motivated student could learn the topic from it with a tutor
beside them: it hooks, checks what they already know, models the method, lets them try,
attacks the classic traps, closes the loop — with zero errors. Grounded in Rosenshine's
*Principles of Instruction*, the Madeline Hunter elements, the 5E model, and the
worked-example effect / cognitive-load theory.

> For the full build patterns — the three nested fades, the canvas-sim recipe, the
> staged-hint mechanic, and the verification cross-checks — read
> **`references/build-method.md`** before building.

## Order of operations (do NOT skip step 1)

1. **Pin scope & level.** Exact topic, grade/curriculum (e.g. Ontario SPH4U, CÉGEP
   203-NYA-05), and how fast to ramp (is the student likely already comfortable?). Ask
   if unclear. If a `create-syllabus` spec exists, align to it.
2. **Find the teaching backbone, then research the content.** Anchor structure on a
   strong canonical lesson (e.g. an Organic Chemistry Tutor video) — note its spine
   (define cleanly → build formulas → worked examples of rising difficulty). Add 1–2
   authoritative content sources (Physics Classroom, OpenStax) and a misconceptions
   source (physics-education research / ERIC).
3. **Write the spine before any HTML:** the ONE big idea, the research-backed
   misconceptions to attack, and 3–5 student-facing objectives.
4. **Build in the fixed structure** (below), using the **Ask → Solve → Watch** rhythm
   per concept.
5. **Verify EVERY number programmatically** (Python, with asserts + physics
   cross-checks) before writing any HTML. No hand-waved arithmetic. See build-method.
6. **Strip all tutor-facing copy.** The material addresses the *student* ("you"), never
   the tutor ("he probably knows," "if shaky, teach," "advance when"). That coaching
   belongs in a separate plan.

## The fixed structure (sections, in order)

1. **Hook / bridge** — connect to what they just learned; include an **animated visual**
   (e.g. a v–t graph whose slope and area light up in sync with the equations). State
   the ONE big idea here. Respect `prefers-reduced-motion`.
2. **Objectives** — "By the end you'll be able to …" (3–5 concrete, checkable skills).
3. **Concept checks (test-as-you-go)** — each idea as a short statement + a *Quick check*
   question + a reveal answer. Lets the tutor skip what's already known.
4. **Method / key equations** — the minimum machinery plus the explicit *selection rule*
   (how to choose which tool to use).
5. **Worked examples — Ask → Solve → Watch, ramped easy → hard.** Each: pose it (Ask),
   work it with **staged reveals** (Solve), and where useful show a **sim/visual**
   (Watch). Group into tiers; difficulty must genuinely climb.
   **Solve opens with the diagram:** the labelled setup sketch a solver draws *first* —
   situation/free-body/motion/component, with every given on it, the unknown marked, and a
   sign convention — before any equation, and the algebra then refers back to it. This is
   Step 1 of every solution (see the **`add-diagrams`** skill).
6. **Misconceptions** — "Watch out for," each trap *named* then corrected.
7. **Closure** — restate the one big idea and how every example was a case of it.
8. **Assigned practice (bottom) — "Try these on your own."** Independent practice the
   student does *after* the session, on their own. Each problem: attempt first, then
   reveal **one step at a time** (staged 3-level hints: nudge → method → math/answer),
   so they pull the *smallest* hint they need. (For a fuller take-home set with
   extension concepts and sister questions, use **create-homework**.)

## Setting the difficulty ceiling (consider — don't mandate)

Where the ramp tops out is a judgment call, not a fixed rule. **Take the real assessment
level into account** — glance at the `create-syllabus` spec's rigor notes, or do a quick
search for what the course's tests actually expect — so the lesson isn't pitched well below
(or absurdly above) where the student is headed. But weigh that against **what's feasible
and appropriate for a first teach**: a lesson's job is to build the method cleanly up to
*solid course level* while managing cognitive load. Leave genuinely test-level-and-harder
stretch to the **homework** (`create-homework`), whose hardest items sit a notch above the
lesson's. In short: let the assessment *inform* the ceiling, not dictate it — and never let
it turn the lesson into a brutal test.

## The staged-hint mechanic (used in Solve and Assigned practice)

One button per problem that cycles through exactly **three** levels, never dumping the
full solution at once:
- **Keep student-facing button labels neutral:** use “Show solution” or “Show next step,”
  never strategy-revealing copy such as “Show diagram-first solution.”
- **Level 1 — nudge:** the idea / which principle, no math — often just "**draw the setup
  and label the givens**" (drawing the diagram is usually the real first move). The
  *revealed* solution then opens with that diagram (via `add-diagrams`).
- **Level 2 — method:** the equation to use and how to set it up.
- **Level 3 — math/answer:** the substitution and the final result.
Colour-code by level; the student reveals only as far as they need. (Multi-part problems
fade *within* the problem too — earlier parts more guided than later ones.)

## Quality checklist (must pass all)

- [ ] Objectives stated to the student.
- [ ] Prior knowledge activated on the first screen (the hook).
- [ ] Animated visual hook present, with a reduced-motion fallback.
- [ ] New material in small steps; every method modelled with a worked example.
- [ ] Every worked solution **opens with a labelled setup diagram as Step 1** (before the
      equations; via `add-diagrams`), and the algebra refers back to it.
- [ ] Checks for understanding throughout (reveal questions).
- [ ] Difficulty genuinely ramps; scaffolds for the hard cases.
- [ ] Misconceptions confronted head-on (named, then corrected).
- [ ] Explicit closure tying back to the objectives / one idea.
- [ ] **Bottom assigned-practice block with staged one-step-at-a-time hints.**
- [ ] Every numeric answer verified in code (with physics cross-checks).
- [ ] Post-build validation: `node --check` the JS; every stepper has exactly 3 levels;
      tags balanced; any sim's `calc()` reproduces the worked answers.
- [ ] 100% student-facing voice; no tutor coaching copy.
- [ ] Clean, consistent notation (subscripts via `<sub>`, units, signs).

## Anti-patterns

- Worked examples with no "you do" → looks complete, teaches passively.
- Tutor instructions leaking into student copy.
- Unverified numbers ("trust me" arithmetic).
- A wall of text with no hook/visual.
- Teaching the right idea without naming the wrong one it replaces.
- A hint button that reveals the whole solution in one click (must stage).
