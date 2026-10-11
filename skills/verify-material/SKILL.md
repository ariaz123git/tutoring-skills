---
name: verify-material
description: >
  Rigorously verify a finished student-facing material (interactive HTML lesson / homework /
  quiz, or a document) before it ships to a student. Runs a GROUND-TRUTH pass first (every
  number re-derived in code, JS syntax-checked, the file rendered and eyeballed, content checked
  against the source/curriculum, tags/structure validated) and only THEN two INDEPENDENT
  adversarial reviewers in parallel — one Opus, one Sonnet — each with full context, a written
  rubric, and an adversarial stance; then synthesizes their findings, revises (capped rounds,
  re-verifying any changed numbers), and stops at a human approval gate. Use when asked to
  "verify," "review before sending," "double-check this lesson/homework," "make sure nothing is
  wrong," "adversarial review," or to gate anything going to a student. NOT for building the
  material (use create-lesson / create-homework / add-visuals / add-diagrams).
allowed-tools: Read Write Edit Bash Agent
---

# Verify Material

A finished lesson or homework can be wrong in ways its author cannot see. This skill is the
gate between "I think it's done" and "a student sees it." It exists because of one stubborn
fact: **a language model cannot reliably find the flaws in its own reasoning by re-reading its
own output.** Improvement needs *external* signal — executed code, a rendered image, the source
document, and independent critics who did not write the thing. This skill supplies all four, in
an order chosen so the cheap deterministic checks catch what they can before the expensive
judgement calls run.

> Reviewer-prompt templates, the rubric template, and the research citations live in
> **`references/reviewer-prompts.md`**. Read it before spawning reviewers.
>
> The mechanics of the render-and-look pass — headless capture, pages taller than the viewport,
> forcing `<details>` open, PDF rasterising and pagination, and the artifacts that make a clean
> file *look* broken — live in **`references/rendering-and-proofing.md`**. Read it before the
> first render. "Render it and look at it" is the step most often skipped, and it is skipped
> because the page doesn't fit in a screenshot.

## Why this shape (the evidence)

- **Models don't self-correct reasoning without external feedback.** Left to "review yourself,"
  an LLM mostly rewrites, not repairs (Huang et al. 2023, *Large Language Models Cannot Self-Correct
  Reasoning Yet*). So the loop must inject outside signal — code output, renders, the source, and
  other models — not just "look again."
- **An independent critic plus a human beats either alone.** A separate critic model surfaces
  bugs the author missed, and critic-assisted humans catch more than unaided humans (OpenAI 2024,
  *LLM Critics Help Catch LLM Bugs* / CriticGPT). Hence: reviewers who did NOT build the material,
  and a human at the end.
- **Judge models are biased** — toward the first option, longer answers, and their own style
  (Zheng et al. 2023, *Judging LLM-as-a-Judge*). Mitigations, all used below: a written **rubric**
  (not vibes), **two different models** (Opus + Sonnet) so one model's blind spot isn't the whole
  review, ground-truth facts fed in as **calibration**, and a human final gate.
- **Accuracy is checked against the source, not memory.** Educational review must verify claims
  against the primary source / curriculum rather than the reviewer's recall (Veletsianos et al. on
  AI-generated educational content). Ground truth comes first so the LLM reviewers are calibrated,
  not trusted blindly.

## Order of operations (do NOT reorder — ground truth precedes judgement)

### 1. Ground-truth layer (deterministic, before any LLM review)
Fix everything found here first; a reviewer's time is wasted on defects a script would catch.

- **Re-derive every number in code** (Python), with **domain cross-checks**, not just arithmetic —
  e.g. recompute a projectile's range from `x = vₓt`, check `v_y = 0` at the apex, confirm a
  discriminant and both roots. Assert the file's stated answers match.
- **Reproduce the *displayed* intermediates, not only the final answer.** A boxed answer can be
  right while the shown line `18.02 × 2.58 = 46.4` is wrong (it's 46.49). Students who calculator-
  check a step and get a different number lose trust in the whole solution. Flag every step whose
  shown arithmetic doesn't reproduce; fix by carrying a guard digit or writing `≈`.
- **Syntax-check the JS**: `node --check`. For interactive widgets, if animation/step logic is
  non-trivial, test in a real DOM (jsdom) and simulate platform quirks (range-input step-snapping).
- **Render and eyeball.** Turn the file (or its diagrams) into images (WeasyPrint → PDF →
  pypdfium2 → PNG, or headless capture) and *look*: clipped labels, colliding text, panels too
  tall, misaligned math, diagram labels that contradict the numbers. **Cover the whole file, not
  the first viewport** — capture a long page in segments, and force every `<details>` open in a
  scratch copy first or the solutions are never seen. **Assume the first render surfaces
  defects** and budget the re-render.
- **A failing check is not proof of a defect.** A hardcoded expected count or a mis-scoped
  substring test is about as likely to be wrong as the artifact. Print the matched text and
  confirm the finding is real *before* editing the material. (SVG's
  `xmlns="http://www.w3.org/2000/svg"` is a namespace identifier, not an external fetch.)
  Equally, do not "fix" the source from a low-resolution render — `²` reads as `³` when
  downscaled; check the source characters first.
- **Check against the source / curriculum**, not memory: right level, right conventions, nothing
  the curriculum forbids, nothing promised-but-missing.
- **Validate structure**: tags balanced, every stepper has its levels, IDs referenced by JS exist,
  no tutor-facing copy leaked into student text.

### 2. Self-review & revise
Re-read the whole thing against the rubric (below) as if you were the student. Fix the obvious
before spending reviewer attention. This is necessary but **not sufficient** — do not stop here.

### 3. Dual independent adversarial review (Opus + Sonnet, in parallel)
Spawn **two** sub-agent reviewers **at once, in a single message** so they run concurrently and
neither sees the other (independence). One on **Opus**, one on **Sonnet** — a two-model ensemble
so a single model's blind spot doesn't sink the review. Each reviewer gets, in its prompt:

- **Fresh context** (a sub-agent, not the builder) and an explicit **adversarial stance**
  ("find everything wrong; over-report rather than miss; do not be agreeable").
- **Full context**: who the student is, curriculum/level, the material's purpose, the history/
  constraints that shaped it (e.g. "worked solutions that read as a terse stack of equations are
  hard to follow alone → follow-ability is the #1 sensitivity"), and **what is intentional** so
  they don't "fix" deliberate design.
- The **ground-truth results** from step 1 as calibration ("final answers are code-verified; here
  are the values; confirm and hunt for what code can't see").
- A **written rubric** (correctness-vs-source first, then student follow-ability, then completeness/
  framing, then visuals/interaction, then notation/leaks).
- **Read the canonical file via the Read tool** — warn them a bash/editor mount can be stale/
  truncated; the Read tool is source of truth. **Report only; do not edit.**
- A **structured output**: verdict {ship / ship-with-fixes / needs-work}, then Critical /
  Should-fix / Nice-to-have (each: quoted location, the problem, a concrete fix), then an explicit
  **coverage confirmation** of what they checked and found correct.

Templates for all of this are in `references/reviewer-prompts.md`.

### 4. Synthesize
Merge the two reports. **Convergent findings** (both reviewers flag it) are highest priority.
Dedupe. For each item decide **fix / won't-fix**, honoring intentional design and stated
constraints (a reviewer suggesting a formula the course deliberately avoids is wrong — note and
move on).
Keep the synthesis; it's the changelog.

### 5. Revise (capped)
Apply fixes. **Re-run the ground-truth checks on anything you changed** — a fix that alters a
number must be re-verified in code. Cap at **≤2 revise rounds**; if issues remain after that,
surface them to the human rather than looping (models can thrash). A second short reviewer pass is
optional only if a Critical fix changed the material substantially.

### 6. Human final gate (never auto-ship)
Present the material to the person (share the file) with a short synthesis: what the reviewers
found, what you fixed, what you deliberately didn't, and any residual judgement calls. **The skill
never ships to a student on its own** — a human approves. This is the CriticGPT lesson: critic +
human, not critic alone.

## The rubric (adapt the specifics per material; keep the order)

1. **Correctness vs. the source** — facts, math, signs, units, every numeric result *and every
   displayed intermediate step*. Highest priority; any error is Critical.
2. **Follow-ability for the actual student** — every step named and motivated; no unexplained
   jump; nothing "messy"; the hardest worked solution scrutinised line-by-line. (Weight this
   heavily whenever the student has struggled with solutions before.)
3. **Completeness & framing** — does it teach/assess what it claims, at the right level, with the
   intended scaffolding; is any tutor-facing coaching leaking into student text.
4. **Visuals & interaction** — diagrams match the numbers and don't collide/clip; animations/
   steppers behave; math renders at normal size.
5. **Notation & consistency** — symbols defined before use, subscripts/signs uniform, no typos.

## Quality checklist (must pass all)

- [ ] Ground truth done **first**: numbers re-derived in code (with domain cross-checks) **and
      displayed intermediates reproduce**; JS `node --check`ed; file rendered and eyeballed;
      checked against source/curriculum; structure/tags validated.
- [ ] Two reviewers, **Opus and Sonnet**, spawned **in parallel** (independent), each with full
      context + ground-truth calibration + rubric + adversarial stance + report-only.
- [ ] Reviewers told to read the **canonical file** (mount-staleness warning) and told what is
      **intentional** so they don't "fix" it.
- [ ] Findings **synthesized**; convergent items prioritised; fix/won't-fix decided against the
      constraints.
- [ ] Revisions applied, **changed numbers re-verified in code**, rounds **capped**.
- [ ] **Human approval gate** before it reaches a student; material shared with a short synthesis.

## Anti-patterns

- Spawning the LLM reviewers **before** the ground-truth pass (they waste effort on script-catchable
  bugs and you trust judgement over facts).
- A **single** reviewer, or two on the **same** model, or reviewers who can **see each other** (loses
  the ensemble/independence that kills single-judge bias).
- "Review yourself and fix it" with no external signal (models rewrite, they don't repair).
- Verifying only the **final answer** while the **shown steps** don't reproduce it.
- Calling the render pass done after **one screenshot of the first viewport**, with the rest of
  the page and every collapsed solution never looked at.
- "Fixing" the material to satisfy a **buggy check**, or to satisfy a glyph misread from a
  downscaled render.
- Letting reviewers read a **stale mount** instead of the canonical file, or letting them **edit**.
- **Uncapped** revise loops; shipping to the student **without a human gate**.
- Treating a reviewer suggestion that contradicts an **intentional constraint** as a required fix.
