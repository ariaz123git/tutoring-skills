# Reviewer prompts, rubric & sources

Reusable templates for step 3 (dual adversarial review). Fill the `‹ ›` slots. Send **both**
reviewer spawns in **one** message so they run in parallel and independently; set one to model
**opus** and one to model **sonnet**. Keep the two prompts identical except the model — identical
prompts make the two reports directly comparable, so convergence is meaningful.

## Reviewer prompt template

```
You are an ADVERSARIAL expert reviewer. Think as hard as you can. Find everything wrong,
confusing, or hard-to-follow in ‹material type› before a real ‹student description› uses it
alone. Be relentless and skeptical, not agreeable. Over-report rather than miss. Do NOT modify
any files — output a written report only.

## What you are reviewing
File (read via the Read tool — SOURCE OF TRUTH; do NOT read it through bash/cat): ‹absolute path›
IMPORTANT: a bash/editor mount of this file may be stale or truncated. Use the Read tool only.

## Full context (so your review is grounded, not generic)
- Student: ‹who, level, curriculum›.
- Purpose of THIS material: ‹what it must do›.
- History / sensitivities: ‹e.g. "worked solutions that read as a terse stack of equations are
  hard to follow alone → follow-ability of worked solutions is the #1 thing to stress-test"›.
- INTENTIONAL — do NOT flag as gaps: ‹deliberate choices, e.g. "derives range from x=vₓt, never
  the range formula"; "diagram-first solutions"; "cold sister problems on purpose"›. If you think
  something is 'missing', check this list first.

## Ground truth already established (code-verified — independently sanity-check any you doubt)
‹paste the step-1 numbers / facts, and any known frictions you already spotted so the reviewer
confirms + extends rather than rediscovers›.

## Your rubric (priority order)
A. CORRECTNESS vs the source — facts, math, signs, units, every result AND every displayed
   intermediate step. Recompute anything you doubt. Any error is CRITICAL.
B. FOLLOW-ABILITY for this student (weight highest if they've struggled before) — every step
   named/motivated; no unexplained jump; the hardest worked solution line-by-line.
C. COMPLETENESS & FRAMING — teaches/assesses what it claims, right level, right scaffolding; no
   tutor-facing copy leaked into student text.
D. VISUALS & INTERACTION — diagrams match the numbers and don't clip/collide; steppers/animations
   behave; math at normal size.
E. NOTATION / CONSISTENCY / TYPOS.

## Output format (report only — no file edits)
1. VERDICT: {ship as-is | ship with fixes | needs-work} + one sentence.
2. CRITICAL (wrong fact/number, or something a student truly can't follow): each = quoted
   location + what's wrong + concrete fix.
3. SHOULD-FIX (follow-ability / clarity / reproducibility gaps): same format.
4. NICE-TO-HAVE (polish): same format.
5. COVERAGE: state explicitly what you checked and found CORRECT (esp. the hardest solutions and
   any structure requirement), so I know what was actually inspected.
Quote text/line numbers. If a category is clean, say so explicitly.
```

## How to spawn (both at once, parallel & independent)

- Emit **two sub-agent calls in a single assistant message** — one `model: opus`, one
  `model: sonnet` — each a general-purpose agent, each report-only. One message = they run
  concurrently and cannot see each other.
- Give each the **same** filled template. Do not let either see the other's output before both
  return.

## Synthesis rule of thumb

- **Both flagged it →** near-certain real; fix unless it violates an intentional constraint.
- **One flagged it →** judge on merits against the rubric and constraints.
- **Contradicts an intentional choice →** note as won't-fix with the reason.
- Re-verify in code any fix that changes a number. Cap at ≤2 revise rounds, then hand residuals to
  the human.

## Why two *different* models (not two of the same)

Judge models share systematic biases (position, verbosity, self-preference — Zheng et al. 2023).
Two instances of one model share that model's blind spots, so their agreement is less informative.
Opus + Sonnet is a small, cheap ensemble: where two different models independently converge, the
finding is robust; where they diverge, you get two lenses instead of one echo.

## Sources

- Huang et al. 2023, *Large Language Models Cannot Self-Correct Reasoning Yet* — self-review
  without external feedback doesn't reliably repair reasoning → inject code/render/source/critics.
- OpenAI 2024, *LLM Critics Help Catch LLM Bugs* (CriticGPT) — an independent critic model + a
  human catches more than either alone → reviewers who didn't build it, plus a human final gate.
- Zheng et al. 2023, *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena* — LLM-judge biases
  (position, verbosity, self-preference) → mitigate with a rubric, model diversity, ground-truth
  calibration, and a human gate.
- Veletsianos et al. — AI-generated educational content must be checked against the source/
  curriculum, not the model's memory → ground truth precedes LLM judgement.
- Pólya, *How to Solve It*; OpenStax University Physics §4 — domain cross-checks for the ground-truth
  layer (e.g. draw-and-label, apex `v_y=0`, range from `x=vₓt`).
