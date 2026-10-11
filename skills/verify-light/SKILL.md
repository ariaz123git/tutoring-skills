---
name: verify-light
description: Token-efficient verification of a finished student-facing lesson, homework, quiz, answer key, or document before delivery. Use when asked to lightly verify, quickly review, sanity-check, or do a one-pass pre-send check of educational material. Runs compact deterministic checks first and exactly one independent Sonnet adversarial review; it never runs an Opus pass or a two-model ensemble.
---

# Verify Light

Verify finished educational material with a small ground-truth pass and one independent Sonnet critic. Keep the output compact and leave final approval to the tutor.

Read [references/sonnet-review-prompt.md](references/sonnet-review-prompt.md) before launching the reviewer.

## Workflow

### 1. Establish the target

Identify:

- canonical material path(s);
- student, course, and expected difficulty;
- source lesson/curriculum to compare against;
- deliberate design choices the reviewer must not misclassify as defects.

Do not rebuild the material during this step.

### 2. Run compact ground-truth checks

Use tools and scripts rather than prose reasoning. Report summaries and failures only; do not paste long logs.

- Recompute every stated answer and important displayed intermediate in code. Include sign, unit, frame, root, and back-substitution checks appropriate to the subject.
- Parse or syntax-check embedded scripts. Validate IDs, balanced structures, and student/key number consistency.
- Inspect the rendered material at representative desktop, narrow, and print states. Check clipping, overlap, legibility, interaction, and answer leakage. Cover the whole file rather than the first viewport, and force every `<details>` open in a scratch copy first or the solutions go uninspected. Mechanics: [../verify-material/references/rendering-and-proofing.md](../verify-material/references/rendering-and-proofing.md).
- Treat a failing check as a claim, not a verdict: read the matched text and confirm the defect is real before editing the material, since a hardcoded count or mis-scoped test is about as likely to be wrong.
- Compare scope, notation, and difficulty with the named lesson and curriculum.

Fix deterministic failures before the model review. Re-run only the checks affected by a fix.

### 3. Run exactly one Sonnet review

Launch one isolated, report-only reviewer with model `sonnet`. Do not launch Opus. Do not launch a second reviewer. Do not ask the builder to impersonate the reviewer.

Pass paths instead of copying whole files into the prompt. Include only:

- student/course context;
- purpose and intentional choices;
- a compact ground-truth summary;
- the rubric and output limits from the reference prompt.

Require the reviewer to read canonical files directly, make no edits, and return no more than 900 words. If Sonnet model selection is unavailable, disclose that and stop rather than silently substituting another model.

### 4. Triage and revise once

Judge each finding against the source and intentional constraints.

- Fix correctness, student-followability, accessibility, or interaction defects.
- Decline taste-only rewrites that do not improve the student outcome.
- Re-verify any changed numbers, diagrams, scripts, or matching student/key text.

Use at most one revision round. A second Sonnet pass is out of scope for this light workflow; unresolved judgement calls go to the tutor.

### 5. Hand off for human approval

Provide:

- verdict: ready, ready with noted caveat, or needs tutor decision;
- files checked;
- concise list of fixes made;
- any residual issue or deliberate non-fix.

Never send material to the student automatically.

## Completion checklist

- [ ] Canonical files and student/course context identified.
- [ ] Answers and displayed arithmetic recomputed.
- [ ] Scripts/structure checked.
- [ ] Representative visual and print states inspected.
- [ ] Scope and difficulty checked against lesson/curriculum.
- [ ] Exactly one independent Sonnet report completed; no Opus pass.
- [ ] Accepted fixes applied and affected checks rerun.
- [ ] Tutor receives the files and a compact approval summary.

