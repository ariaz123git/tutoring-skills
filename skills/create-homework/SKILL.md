---
name: create-homework
description: >
  Build a student-facing take-home assignment (interactive HTML) the student does ALONE
  after a lesson — attempt-first, with adaptive part-aware hints, plus new
  application/extension content not covered in the lesson and "sister" practice that
  mirrors the lesson's examples. Use when asked to make "homework," an assigned practice
  set, or take-home work to consolidate a taught topic. It is test-aware — it researches
  what the student's real assessment looks like and calibrates the hardest items to (and a
  notch above) test level. Grounded in the research on effective homework (retrieval
  practice, spacing, interleaving, the ten-minute rule, feedback). NOT for in-session
  teaching material (use create-lesson) or a course map (use create-syllabus).
allowed-tools: WebSearch WebFetch Read Write Edit Bash
---

# Create Homework

Homework that works isn't a worksheet dump — it makes the student **retrieve from
memory**, **stretches** them just past the lesson, and gives **feedback only as needed**
so they build independence. (Secondary-school homework has a ~+0.64 effect size *when
it's short, focused, and purposeful* — long busywork doesn't.)

> Shares the **staged-hint mechanic** and the **verification workflow** with
> `create-lesson` — see its `../create-lesson/references/build-method.md` for the full patterns.

## Order of operations

1. **Anchor to the lesson — in the student's own language.** Read the lesson file / log so
   homework *extends* rather than repeats, and reuse the class's **exact vocabulary** (see
   *Meet the student where the lesson left them*). When you must assume a term, notation,
   prerequisite, or difficulty, **state the assumption in the build notes so the tutor can
   correct it** — most bad homework traces back to an un-surfaced wrong assumption.
2. **Decide the extension.** Pick 1–2 **application concepts the lesson didn't cover** —
   a real new bit of learning the student can pick up semi-independently (taught briefly
   in the homework with a worked example), not just harder versions of the same problem.
3. **Calibrate difficulty to the real test** (see *Test-awareness* below) — research what
   the student's actual assessment looks like and set the hardest-question target from it.
4. **Verify EVERY number in code first**, with **domain cross-checks** — not just arithmetic
   (see *Verification*).
5. **Build in the fixed structure** (below), student-facing voice only.
6. **Keep it short.** The ten-minute rule caps *total* nightly homework across all subjects
   (~10 min × grade level), so a single subject's set is only a slice of that — aim for
   roughly **15–25 min** (a bit more only if it carries a synthesis capstone), focused.
7. **Validate & proof** — structural checks, **render every visual output to images**, and
   **test any interactive JS in a real DOM** (see *Verification*).
8. **(Optional) extra deliverables.** If the topic benefits from diagrams, layer in
   **attempt-first visuals** (hidden behind a toggle; see the `add-visuals` skill). If the
   student wants to work on paper or an iPad, also produce a **print/PDF variant** (see
   *Output formats*).

## The fixed structure (sections, in order)

1. **Instructions** — do this on your own; **try each before revealing anything**; reveal
   **one step at a time** and stop as soon as you're unstuck; the sister questions are
   meant to be done *without* peeking (but you can if you must).
2. **Warm-up retrieval** — 2–4 quick recall items answered **from memory, no notes**.
   Retrieval, not re-reading, is what builds retention. But **only quiz what the student was
   expected to know cold** — anything provided on the real assessment (a formula sheet,
   constants table) becomes *conceptual* recall (when it applies, how to choose it, what it
   leaves out), never "reproduce it from memory" (see *recall-fairness* below).
3. **New application (extension)** — the 1–2 concepts the lesson didn't cover: taught as a
   proper **mini-lesson** (short teaching box + worked example + a couple of problems), not
   just harder versions. This is where the student learns something new semi-independently —
   see *Teaching new material* for how to teach it (derive, build notation, full reasoning).
4. **Sister practice** — variations that mirror the lesson's worked examples, **unguided**
   (a plain full solution to reveal, no staged drip), to prove they can do it cold. Pitch
   the hardest sister(s) at the student's real **test level**, with the top one a notch
   above (the train-harder buffer); keep the rest at or below so success stays high.
5. **Spaced/interleaved review** — 1–2 problems from an *earlier* topic, mixed in, so the
   assignment isn't all one type (spacing + interleaving both improve long-term recall).
   *Optionally cap this tier with one **synthesis capstone**:* a single hard, exam-style
   question that fuses the current topic with a prior one and forces **multiple
   representations** (e.g. solve a multi-phase motion *and* draw its d–t / v–t / a–t
   graphs). Give it a full worked solution — including a correct-answer figure to check a
   drawn graph against. This is the one item deliberately set a notch above test level.
6. **(Optional) self-explanation prompt** — "in one sentence, why does this method work?"

## Meet the student where the lesson left them

The biggest source of bad homework is the generator **assuming** something — a term, a
notation, a prerequisite, a difficulty — that doesn't match what this student was actually
taught. Anchor to the lesson/log; when you must assume, surface it for the tutor.

- **Use the class's exact vocabulary.** Mirror the lesson's wording verbatim; don't
  substitute textbook-standard names for what the class calls things. Wrong terminology
  quietly signals "this wasn't made for you."
- **Recall-fairness.** A warm-up may only ask for what the student was expected to know
  *cold*. Anything handed to them on the real test is **not** a memorization target — quiz
  understanding of it, never reproduction.
- **Don't give away structure the student is meant to recognize.** If identifying the
  approach *is* the skill (which method, how many stages, which representation), don't label
  it on the practice/cold items — recognizing it is the point. Worked/demo items may show
  structure; cold items must not.

### Teaching new material (the mini-lesson pattern)

Extension content the lesson didn't cover must be *taught*, not dropped in harder. Shape it:
teaching box (what's new + the tool + one or two lines **deriving** it) → optional
prerequisite explainers → a worked example → the problems with staged hints → a signpost
that this is new so struggle is expected. Within that:

- **Notation component-first, then compact.** Compute the explicit pieces first, then
  introduce any shorthand and say what each slot means. New notation is earned, not assumed.
- **Derive, don't cite.** Get the working equation from a principle the student already
  knows; never present a rearranged formula as if it fell from the sky.
- **Every hint advances one distinct reasoning decision.** Use the smallest useful prompt:
  an informational clue, a Socratic checkpoint, or targeted misconception feedback. Do not
  force a nudge/method/math sequence or split routine algebra into artificial microsteps.
  Narrate silent leaps in the final solution, not by leaking them prematurely through hints.
- **Prerequisite explainers.** New topics silently depend on sub-skills (reading a bearing,
  a sign convention, a graph axis). Attach an **optional, pressable, hidden-until-clicked**
  helper right where the dependency appears — it teaches the *background* skill (unlike a
  hint, which helps solve *this* problem) without cluttering the page for those who don't
  need it.

### Restraint & references

- **The statement poses the problem, not the strategy.** Strategy hints live behind the
  reveal, pulled only if stuck. No "(don't assume — let the signs tell you)" giveaways.
- **Keep reveal labels neutral.** Use “Show solution” or “Show next step”; never advertise
  the intended strategy with labels such as “Show diagram-first solution.”
- **No cryptic citations.** Gloss any cross-reference in plain language ("that's the rule in
  the reference box up top"), never a bare code the reader has to hunt for.
- **No editorial/version leaks.** A problem reads as itself — never "no graph this time,"
  "the harder version," "now uses…," or QA notes like "all answers verified."
- **Surface a second method** when one exists — especially a graphical one that reuses a
  representation the student just built ("or, straight off your x–t graph…").
- **Link the tutor's actual resources** (the specific videos they used) contextually, near
  the relevant section — get the real URLs from the tutor; don't guess or invent them.

## Mechanics

- **Adaptive, part-aware hints** on application and harder items. A part may need no hint,
  one strategic clue, or several linked prompts when it contains several genuine reasoning
  bottlenecks. Attach each hint to the part it supports and fade help across later parts.
  Read `references/hint-design.md` before designing interactive hints.
- Default learned, routine, cold-check work to **no proactive hint**. Do not invent a hint
  ladder simply because the format supports one; targeted feedback may follow a real error.
- Keep **hint**, **hint answer**, **final answer**, **full solution**, and **prerequisite
  explainer** distinct. A hint helps the next decision; a hint answer resolves only its small
  checkpoint; a final answer reports the result; a full solution explains the whole chain;
  an explainer teaches background knowledge without solving the current problem.
- **Sister questions** get a single plain "Show solution" reveal (no staging) — they're
  the cold check.
- **Attempt-first by default:** everything hidden until the student acts; nothing
  auto-reveals.
- **Visuals stay attempt-first too** (toggle-gated, demo-items-only) — see *Visuals*.

## Solutions read as explanations (diagram-first, follow-able)

A solution the student **can't follow teaches nothing** — the commonest failure is a reveal
that's just a terse stack of equations that reads as "messy." Every solution reveal must be a
clean, labelled **explanation a student who is shaky on the topic can follow on their own**:

- **Open with a diagram when a representation materially helps solve the problem** — for
  example a situation sketch, free-body diagram, graph, sign chart, number line, or component
  triangle. Put it before equations and refer back to it. Skip decorative diagrams for direct
  recall, routine algebra, and one-step substitutions (see the **`add-diagrams`** skill).
- **Then walk it in named steps**, each saying *what* you do and *why* in plain language — not
  a wall of symbols. Show the substitution, then the result; label the final answer.
- **Assume no prior knowledge of the topic.** Where a step leans on a concept the student may
  not have (a formula, a technique, a sign convention), **link to a short explanation** (a
  pressable inline explainer or a hyperlink) rather than assuming it.
- **One idea per line, clean notation.** Break a long derivation into short lines; don't cram.

The test: seeing only the diagram + steps, could the student follow every move without you
there? If a line would make them ask "where did that come from?", add the sentence or the link.

## Output formats

- **Primary: interactive HTML** — the attempt-first reveals and any visuals live here.
- **Optional print/PDF (iPad-writable)** — a *static* twin of the same content, for a
  student who'd rather write by hand or annotate on a tablet. Rules that keep it clean:
  - **Build the twin *from* the interactive file, not alongside it.** Parse the built HTML and
    re-emit it in print form, so the two cannot drift when either is edited. When carving blocks
    out, **match tags by depth, not with a non-greedy regex** — `<div class="card">(.*?)</div>`
    stops at the first *nested* close and silently returns a truncated block while the match
    count still looks right. Assert every extracted block still contains what it must (its
    solution, its figure). Helper in `../verify-material/references/rendering-and-proofing.md` §8.
  - **Worked/demo items show their solution inline** (they teach); **attempt-first items
    (your-turn, sisters, capstone, extra) get modest, quiz-like work space** — not
    generous — plus a **blank labelled grid** wherever a question asks for a drawn graph.
  - Put **all attempt-first solutions in one "Solutions" section at the end** (page-break
    before it) so the student attempts first, then flips to check — mirror the interactive
    reveals. Include the **correct-answer figure** for any graphing question. Anything the
    interactive version hides behind a *nudge* belongs here too, ahead of the full solution, so
    the print reader can still take the smallest help first.
  - Strip everything interactive (buttons, sliders, JS); add a name/date line and the
    given constants at the top. Assert the emitted twin contains no `<script>`, `<button>`,
    or `<details>` — a stripping step that silently half-works is easy to miss.
  - **`break-inside: avoid` is not free, and it fails in both directions.** Too little and a
    work box orphans onto the next page away from its problem; too much and every block that
    won't fit in the page's remainder jumps to a fresh page — the pathological case is one
    problem per page with half of each page blank. Apply it to things that fit on a page
    (figures, tables, callouts, problem blocks); use `break-after: avoid` on headings, step
    labels, and figure captions; leave long solution bodies breakable. Then **measure the real
    rendered block heights** and size the repeating block so a whole number of them tiles the
    page (US Letter at 15 mm margins ≈ 947 px of content) — usually by trimming work space,
    which should be modest anyway. **Check the page count against your estimate** in both
    directions. Method: `../verify-material/references/rendering-and-proofing.md` §4.
  - **Generate it HTML→PDF with WeasyPrint** (`pip install weasyprint`), using a Unicode
    font (DejaVu Sans/Serif) so math glyphs (√ Δ × − ½ ² ⇒) render. Use `<sub>`/`<sup>`
    for subscripts/superscripts. (If you use ReportLab instead, **register a Unicode TTF
    (DejaVu Sans) with `registerFont`** — then every glyph, incl. ² ₀ − ° ·, renders
    natively; only with ReportLab's *built-in* fonts do Unicode sub/superscripts become
    black boxes, and there you must use its `<sub>`/`<super>` markup.) **If neither is
    installed**, headless Chrome `--print-to-pdf` works and embeds subsetted fonts, so Unicode
    math renders correctly — see the proofing reference.
  - **Proof it visually:** render the PDF pages to images and check for clipped axis
    labels, colliding tick/axis text, and awkward page breaks before shipping. If no PDF
    rasteriser is installed, proof the **print HTML** in segments and check the PDF
    structurally (page count, embedded fonts).

## Visuals (static diagrams & interactive)

- **Static diagrams — size for clarity, not compactness.** If angles, components, or
  geometry matter, give the figure room; show the reference frame (axes / compass key);
  label angles with an **arc** from the reference axis (not a floating number); caption
  every figure in one plain sentence; use a right-triangle sketch wherever components
  combine into a magnitude+direction. **Compute geometry in code** (arc endpoints,
  vertices), don't eyeball it.
- **Label values in-scene, on the thing they measure** (default) — the radius carries its
  `r = …`, a vector is tagged with its value at the tip, an angle shows its value on its arc —
  rather than only in a separate readout box. In an animated scene the labels update live.
- **A worked/demo solution reveal includes a human-style labelled setup diagram when that
  representation aids the setup** — for example a scene, graph, sign chart, or vector view
  with relevant givens tagged in place. Skip it when it would be decorative. (See `add-visuals`
  `patterns.md` §7a–7b.)
- **Interactive visualizations are attempt-first and demo-only** — toggle-gated, hidden
  until pressed, built lazily on first open, and only on worked/demo items (never cold ones).
- **If a visualization animates, avoid these bugs** (they otherwise ship silently — the
  `add-visuals` skill owns the depth, but at minimum):
  - **Never use an input widget's value as the animation clock.** A range slider *snaps its
    value to its step*, which breaks "restart from the end" and makes motion stutter. Keep
    an internal floating-point clock; write to the control for display only.
  - **One uniform time-rate across segments/phases** — don't clamp each segment's duration
    independently, or short segments crawl and long ones race.
  - **Multi-panel SVGs need a flex wrapper with capped widths and a `max-height`**, or a
    near-square panel balloons to full-width-tall (unrenderable, laggy).
  - **Guard the animation loop** (cancel before starting; reset cleanly so replay works).
- **Render math at normal body size.** Oversizing inline math misaligns radicals, inflates
  sub/superscripts, and clashes with the text. Let long equations wrap, and **match an
  existing good file** rather than re-inventing math styling.
- **Build a real radical.** The monospace "equation" face draws `√` as a bare rising stroke with
  **no bar over the radicand** — at body size it reads as a capital `J`, and `√(a² + b²)` loses
  the one cue that shows how far the root extends. Take the sign from a math face and draw the
  bar with a `border-top` over the radicand; convert at build time (matching the radicand's
  parentheses **by depth**) and assert no bare `&radic;` survives. Everything else
  (`− × ° α θ φ → ≤ ² ₀`) renders fine in the monospace face. Recipe:
  `../verify-material/references/rendering-and-proofing.md` §4b.

## Verification (ship no bugs)

1. **Every number in code first, with domain cross-checks** — not just arithmetic. Confirm
   the result satisfies the governing relationships (a conserved quantity balances; a value
   that can't increase doesn't; a meeting point agrees from both objects' equations; an
   answer back-substitutes into the original equation). A wrong "answer" is worse than none.
2. **Structural validation:** scripts parse (`node --check`); variable-length hint sequences
   reveal in order; dependencies unlock correctly; cold/test mode exposes no coaching; tags
   balance; interactive answers match verified numbers; no student-name/editorial leaks.
3. **Render every visual output to images and look at it.** Many defects are *visual only*
   and invisible to logic checks — clipped labels, colliding text, oversized/misaligned
   math, panels too tall. Preview a render (WeasyPrint, or a headless capture) before shipping.
   **Assume the first render is wrong and budget a second**; proof figures on their own page
   before they go into the material; expand every `<details>` in a scratch copy first or the
   collapsed content is never seen. Mechanics — capturing a page taller than the viewport,
   measuring its height, PDF rasterising, and why a downscaled render turns `²` into `³` — are
   in **`../verify-material/references/rendering-and-proofing.md`**.
4. **Test interactive JS in a real DOM (e.g. jsdom) and simulate the platform's quirks.**
   Logic-only reasoning and naive DOM tests pass code the real browser breaks — model the
   quirks that matter (input widgets that snap/clamp values, event timing). Drive animations
   frame-by-frame with a mocked clock; assert smooth progression, correct restart, correct
   end state.
5. **Tooling gotcha:** the file you edit is the source of truth; a shell/cache view can lag
   and show a stale or truncated copy (phantom "syntax errors," wrong line counts). Verify
   against the authoritative file layer; when you must test in the shell, write a fresh copy
   the shell owns rather than reading back through a mount. Write generator/validator scripts
   with the **file tools, not shell heredocs** (heredocs mangle escapes and quoting, producing
   "syntax errors" that are shell artifacts), and set `PYTHONIOENCODING=utf-8` for any script
   that prints `− × √` on Windows.
6. **When a check fails, verify the check before editing the file.** A hardcoded expected count
   or a mis-scoped substring test is about as likely to be wrong as the artifact. Print the
   matched text, confirm the finding is real, then fix whichever is actually broken. (SVG's
   `xmlns="http://www.w3.org/2000/svg"` is a namespace identifier, not an external fetch —
   whitelist it before flagging "external URLs".)

## Test-awareness (calibrate to the real assessment)

Homework should look like the test it's preparing for — and the top of it should be a
little harder. Web-search for what the student's actual assessment looks like for this
topic (sample unit tests, past exams/finals, question style, format, and the difficulty
of the hardest items), then **calibrate** to it. Three guardrails keep this honest and useful:

1. **Calibrate, then generate original.** Read difficulty and style off real exams, then
   write *new* questions at that level. Never copy real test items verbatim — copyright,
   and they won't be the student's actual test anyway.
2. **Be honest about what's findable.** Where standardized exams exist (CÉGEP finals, AP,
   IB, some boards) calibration is strong. Where tests are teacher-made and not public
   (e.g. Ontario SPH4U), you're approximating from *representative* samples — say so in
   the build notes; don't claim you found "the test."
3. **A component, not the whole.** Test-aligned items live in the harder/sister tiers, not
   the entire assignment — the retrieval / extension / spacing structure still does the
   teaching. Set the **hardest one or two questions a notch above test level** as a
   deliberate "train harder than you fight" buffer; keep the rest at or below so the
   success rate stays high.

## What research says (bake in, don't pad)

- **Retrieval practice** — recalling from memory (vs. re-reading) is the strongest driver
  of retention → lead with no-notes recall.
- **Spaced practice** — revisiting material after a gap beats massing it → include older-
  topic items.
- **Interleaving** — mixing problem types improves transfer → don't make every item the
  same shape.
- **Feedback** — make it available but **earned** (staged, attempt-first); for self-study,
  the student getting the *minimum* hint needed mimics good tutoring feedback.
- **Ten-minute rule / purpose over volume** — short, focused, clearly purposeful work
  outperforms long sets; cut anything that's busywork.

Sources: retrieval/spacing/interleaving & feedback research (CIRL/Eton; SecEd; Third Space
Learning); Cooper's homework research / the ten-minute rule; Hattie effect-size data.

## Quality checklist (must pass all)

- [ ] Anchored to a specific lesson; extends rather than repeats it.
- [ ] Opens with from-memory **retrieval** items.
- [ ] Teaches 1–2 genuinely **new application** concepts with a worked example.
- [ ] **Sister practice** mirroring the lesson (unguided, plain-solution reveal).
- [ ] At least one **spaced/interleaved** older-topic item.
- [ ] Hardest items **calibrated to the real assessment** (original questions; honest about
      whether the test is standardized or only approximated); top 1–2 a notch above test level.
- [ ] Adaptive hints on taught/harder parts: each advances a distinct decision, none leaks a
      later answer, dependencies work, and the amount of help matches the actual bottlenecks.
- [ ] Attempt-first; nothing reveals automatically.
- [ ] Short and focused (ten-minute-rule sized).
- [ ] 100% student-facing voice; clean notation.
- [ ] Every solution reveal is a follow-able, named-step **explanation** (not a terse equation
      dump); it opens with a labelled diagram when that representation helps, and assumes no
      topic knowledge — any leaned-on concept is linked to a short explainer.
- [ ] Uses the class's own **terminology**; no un-assigned recall (formula-sheet items →
      conceptual recall); no assumed prerequisite left unexplained.
- [ ] New material **taught** (notation component-first, derived not cited, full reasoning at
      every hint level, pressable prerequisite explainers) and signposted as new.
- [ ] Statements pose the problem only — no strategy giveaways, no structure labels the
      student should recognize, no cryptic citations, no editorial/version leaks.
- [ ] Second/graphical method surfaced where one exists; tutor's real resource links (real
      URLs) placed contextually.
- [ ] Figures clear (sized; axes/compass; arc-labelled angles; captions; triangles for
      component combination; geometry computed in code).
- [ ] Numbers pass **domain** cross-checks; every visual output **rendered to images and
      eyeballed**; any interactive JS **tested in a real DOM with platform quirks simulated**.
- [ ] Any animated visualization avoids the known bugs (internal clock not a stepped input,
      uniform rate, flex-wrapped size-capped panels, guarded loop); math at normal size.
- [ ] *(If a PDF was requested)* static twin with quiz-like work space, blank grids for
      any graphing question, solutions section at the end, and PDF proofed as images.

## Anti-patterns

- A long set of near-identical problems (no retrieval, no interleaving, busywork).
- Repeating the lesson's exact examples with new numbers and calling it extension.
- One-click full-solution reveals used as the only support on taught/harder items; provide
  the smallest useful part-level hint first, then keep the complete solution separate.
- No new learning — homework should move the student forward, not just drill.
- Copying real exam questions verbatim, or claiming you found "the test" when it's a
  teacher-made one you can't actually see.
- Calibrating the *whole* assignment to test difficulty (it crowds out retrieval, spacing,
  and the extension learning).
- Renaming concepts to textbook-standard terms the student's class doesn't use.
- Quizzing memorization of what the assessment hands out on a sheet.
- Parachuting in new notation or a rearranged formula without building/deriving it.
- Pre-loading strategy in the statement, or labelling the structure the student is meant to
  recognize; cryptic cross-references; editorial/version leaks.
- Driving an animation off a stepped input widget; shipping any visual without rendering it
  to look, or interactive JS without testing it in a real DOM.
- A solution reveal that's just stacked equations — no opening diagram, no words explaining
  each move, no link for a concept it leans on (the "messy, couldn't follow it" failure).
- Unverified answers (arithmetic only, no domain cross-check).
