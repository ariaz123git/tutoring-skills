---
name: create-syllabus
description: >
  Reconstruct an accurate, course-specific syllabus / curriculum spec for a student
  who is preparing (often over the summer) for a course they have NOT started yet and
  so has no teacher handout. Use when given target-program context — an Ontario high-
  school course code (e.g. SPH4U), a Quebec CÉGEP course + competency (e.g. 203-NYA-05
  / 00UR), a university department course, or an AP/IB subject — and asked to map the
  real curriculum the student will face. Produces a unit-by-unit spec (objectives, key
  equations, assessment model, sources) that also doubles as a tutoring reference with
  status tracking and gap analysis. NOT for an instructor writing their own syllabus
  from scratch, nor for building the lessons/practice themselves (use create-lesson /
  create-homework).
allowed-tools: WebSearch WebFetch Read Write Edit Bash
---

# Create Syllabus

Build the syllabus the student *will* meet, from the authority that defines it — not a
vague topic list. Because it's summer, the teacher's specific outline does not exist
yet, so you reconstruct from the **governing curriculum** and flag what varies locally.

## The method (do these in order)

1. **Identify the exact course + its governing authority.** Pin the course code and the
   document that legally/officially defines its content:
   - **Ontario high school** → course code (SPH4U, MHF4U, …). Authority: *The Ontario
     Curriculum* (gov of Ontario) + the specific board/school course outline.
   - **Quebec CÉGEP** → course number + **competency code** (203-NYA-05 / 00UR).
     Authority: college *plans de cours* + the program competency document.
   - **University** → department + course number. Authority: the department's posted
     syllabus / academic-calendar entry.
   - **AP / IB** → the College Board *Course & Exam Description* / IB *subject guide*.
   If the student gave only "board + grade + subject," infer the code, then confirm.

2. **Find authoritative sources and triangulate at least two.** Prefer the governing
   curriculum document first; corroborate unit ordering and weighting with 1–2 *real*
   course outlines from comparable institutions. Do **not** rely on a single tutoring
   blog as the source of truth (use them only to cross-check).

3. **Extract the canonical structure:** strands/units → topics → key equations → typical
   problem types → teaching sequence → assessment model → prerequisite → math level →
   optional/teacher-dependent topics.

4. **Reconstruct, don't invent.** Build from the authority. Where school/instructor
   choices vary (exact weightings, dates, optional topics, unit order), say so plainly
   and tell the reader to confirm against the real outline once the term starts. Never
   present a specific school's weighting or schedule as if it were fixed fact.

5. **Build the spec in the fixed format** (below).

6. **Verify.** Every equation correct; topics/units actually match the curriculum;
   cite each source with a link; list assumptions explicitly.

7. **Make it a tutoring instrument.** Add a status legend, a materials index, and a gap
   analysis so the spec doubles as the working map for generating aligned lessons,
   practice, and homework — and for tracking what's done vs. outstanding.

## Fixed output structure

- **Header:** title · student / board-or-institution / course · **Purpose** · a
  "summer / no outline yet → reconstructed from the curriculum" context note.
- **§1 Course at a glance** (table): code & title · credit/hours · prerequisite ·
  governing curriculum · typical textbook · math level required · assessment model ·
  grading standard / pass mark.
- **Strands / competencies list** + **typical teaching order** + one **key scheduling
  insight** (e.g. "the course assumes the prior year is mastered and opens at X").
- **§2 How to read** (status legend: ✅ built · 🟡 partial · ⬜ not built · ⭐ optional).
- **§3 Unit-by-unit:** each unit → *strand mapping · objectives · key equations ·
  typical problem types · rigor & links to prior course · status & our materials.*
- **§4 Materials index** (what's built, where).
- **§5 Gap analysis** in teaching order (what's next, prioritized).
- **§6 Sources** (with links) + a variability caveat.

## What research says a syllabus needs (bake in, don't pad)

Effective syllabi state **measurable learning outcomes**, a **topic/assessment schedule**,
an **assessment plan aligned to the outcomes**, **resources**, and read as **clear and
learner-centred** (MIT Teaching + Learning Lab; Duke CTL; syllabus-design literature).
For a *reconstruction* specifically, the two top priorities are **fidelity to the
governing curriculum** and **transparency about assumptions** — accuracy beats polish.

## Quality checklist (must pass)

- [ ] Exact course code and governing authority identified.
- [ ] ≥ 2 authoritative sources triangulated; all links cited.
- [ ] Every unit has objectives + key equations + typical problem types.
- [ ] Assessment model and grading standard stated.
- [ ] Prerequisite course and required math level stated.
- [ ] Optional / teacher-dependent topics flagged as such.
- [ ] Explicit "confirm against the real outline" caveat.
- [ ] Doubles as a tutoring reference (status legend + materials index + gaps).
- [ ] No invented specifics presented as fact (no fabricated weightings or dates).
- [ ] Every equation verified correct.

## Anti-patterns

- Treating a tutoring-blog summary as the authoritative source.
- Stating one school's weightings/dates as if universal.
- A flat topic list with no equations, no sequence, no assessment model.
- Omitting sources or hiding the assumptions.
- Over-promising precision the curriculum doesn't fix (presenting variable choices as settled).
