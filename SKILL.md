---
name: lesson-plan-builder
description: Builds one complete, teachable lesson plan - a measurable objective, a timed four-part arc (hook, direct instruction, guided practice, closure), a formative check with a numeric proceed/re-teach rule, and differentiation notes - clear enough that a colleague could run it without asking a question. Ships an executable quality gate (scripts/validate_plan.py) that hard-fails the classic failure modes instead of describing them. Use when someone asks "write a lesson plan for...", "plan a 50-minute class on photosynthesis", "how should I structure tomorrow's lesson", "turn this standard into a lesson", or "review/check this lesson plan". Do NOT use for sequencing a multi-week unit or full course - use curriculum-mapper instead; for deep per-learner adaptation of an existing lesson, use differentiated-instruction; for writing the quiz itself, use quiz-generator.
agent_created: true
version: 1.0.0
---

# Lesson Plan Builder

A strong lesson plan is a contract between teacher and learners: here is where we start, here is what we will do, here is how we will know we arrived.

The costly failure this skill prevents is the lecture-heavy plan. The teacher talks for 30 minutes, practice gets squeezed into the last 5, and the first real evidence of confusion arrives on the graded test — when it is too late to act. Every rule below exists to make that plan impossible to write by accident.

## The one idea that organizes everything

**Backward design.** Objective → exit task → activities, in that order.

If you choose the activity first, the activity smuggles in its own objective. A "fun sorting game" quietly sets the target at *remember the categories*, whatever the objective claims. Working in order is not tidiness; it is the mechanism that keeps the lesson measuring what it says it measures.

## Operating procedure

Work the steps in order. The objective must exist before the exit task, and the exit task before the activities.

### Step 1 — Gather inputs

Collect before drafting. Where the user cannot answer, use the default and **label the assumption as a guess** (it goes in the `assumptions` array and is rendered visibly in the output).

1. Subject and topic (required).
2. Grade or level (required — it sets vocabulary and pacing).
3. Class length in minutes (default 50, but confirm it for *this* period; assemblies move it).
4. What students already know — prior lesson or prerequisite skill.
5. Standards to align, if any (default: none, omit the row).
6. Constraints: materials, tech, class size, known student needs.

### Step 2 — Write the objective first

Form: **Students will be able to [observable verb] [specific content] [condition or standard].**

- Pick the verb from `references/blooms-verbs.md`. Reject vague verbs — `understand` and `appreciate` cannot be observed and are on the banned list.
- Name a condition (*using the diagram*, *without notes*, *in two sentences*, *independently*). An objective with no condition has no finish line.
- One primary objective per lesson. A second is allowed only if labelled `Secondary:`.

`references/objective-patterns.md` has the canonical form, rewrite drills for weak objectives, and the anti-patterns.

### Step 3 — Design the exit task before any activity

Write the summative check now, and **mirror the objective's wording**. If the objective says "compare two sources using textual evidence", the exit task asks students to compare two sources using textual evidence — not to list the traits of each source. A task that swaps in a cheaper verb measures the cheaper skill, and the gap surfaces on the graded test.

`references/exit-task-patterns.md` gives the smallest faithful task form for each verb family.

### Step 4 — Build the four-part arc with a time budget

Do not estimate the minutes. Compute them:

```bash
python scripts/budget.py --duration 45        # one lesson
python scripts/budget.py --range 20 90 --step 5 # reference table
```

The three constraints behind the numbers:

- **Hook 3–7 min.** Activate prior knowledge or pose a productive puzzle. Past 7 minutes it is eating practice time.
- **Direct instruction ≤ 15 min, hard ceiling.** Attention is not culturally calibrated and it is not negotiable by subject. If the content cannot be modelled in 15 minutes, that is two lessons, not a longer lecture.
- **Practice ≥ 50% of the lesson** (guided + independent). Students doing, not watching.
- **Closure 3–5 min.** Students synthesise ("what changed in your thinking?"), not merely summarise.

`references/time-budget-table.md` has the derived table for 20–90 minutes, the two-cycle structure for double periods, and the common misallocations.

### Step 5 — Filter activities against the objective

For each candidate activity ask: *if a student does this well, are they closer to meeting the objective?* If no, cut it — engagement alone does not qualify. Prefer activities that generate evidence you can see from the back of the room: mini-whiteboards, think-pair-share, exit slips.

`references/activity-library.md` lists activities by evidence generated and by arc phase, plus the ones that reliably fail the filter.

### Step 6 — Place the formative check with a decision rule

One mid-lesson check at minimum, positioned after guided practice begins and before independent practice. It needs **all three** parts:

1. A **visible whole-class instrument** (not a show of hands — you cannot tell a guess from a reason).
2. A **numeric threshold**: "≥80% of boards correct", "at least 24 of 30".
3. A **named re-teach move** you could start in ten seconds — a second worked example, a peer explanation round, contrasting correct vs incorrect work. "Review it" is not a move.

`references/formative-check-design.md` covers each part, where to place the check, and why procedural lessons want two.

### Step 7 — Add materials and differentiation

List only materials actually required; flag anything needing advance preparation. One scaffold for students who need support, one extension for early finishers — **brief notes, not separate plans**.

The rule that matters: **reduce language load, never reasoning load.** Simplifying the instructions is scaffolding; simplifying the thinking has changed the objective.

`references/differentiation-scaffolds.md` orders scaffolds strongest to weakest and keeps extensions on the objective.

### Step 8 — Run the gate

```bash
python scripts/validate_plan.py plan.md          # human report
python scripts/validate_plan.py plan.json --json # machine-readable
python scripts/validate_plan.py plan.md --min-score 90
```

Exit code is 0 when nothing hard-fails and the score meets `--min-score` (default 75), so this can gate a generation loop. Repair the highest-weight failure and re-run; do not patch a low score by adding words.

## Working with the scripts

Build the document from a plan JSON rather than hand-writing the Markdown:

```bash
python scripts/build_plan.py --in plan.json --out plan.md --check
python scripts/build_plan.py --in plan.json --duration 45   # override the length
```

**Leave a segment's `minutes` as `null` and the budget supplies it.** Explicit times are respected as written, and the validator reports it if they do not add up. `--check` validates the *resolved* plan, so inherited times are judged, not reported as missing.

| File | Role |
|---|---|
| `scripts/lesson_lib.py` | Shared logic: verb banks, budget, Markdown parser, the fourteen-check rubric |
| `scripts/budget.py` | Duration → four-part time budget |
| `scripts/build_plan.py` | Plan JSON → lesson plan Markdown |
| `scripts/validate_plan.py` | Plan (JSON or Markdown) → scored report, gateable exit code |

Standard library only, Python 3.9+. No install step.

### The fourteen checks

`references/quality-rubric.md` is the authoritative table; `scripts/validate_plan.py` implements it exactly — if you change a weight in one, change it in the other. Five checks carry half the score because each maps to a documented failure mode: exit task mirrors the objective (12), direct instruction ≤ 15 min (12), practice ≥ 50% (12), numeric threshold (10), named re-teach (4).

Be honest about the boundary. The validator reads text, not classrooms. It cannot tell you whether the assumed prerequisite is actually in place, whether the threshold is calibrated for *this* class, or whether the content is at the right difficulty. Those remain judgement — the rubric just clears the mechanical failures out of the way so your judgement can spend its budget on the ones that are not mechanical.

## Templates and worked examples

| File | Use |
|---|---|
| `assets/templates/plan.schema.json` | Input contract for `build_plan.py`; read it before hand-writing JSON |
| `assets/templates/lesson-plan.md` | The output layout, with placeholders |
| `assets/templates/prep-checklist.md` | Pre-lesson checklist, ordered so anything needing a printer happens first |
| `assets/examples/01-biology-photosynthesis.plan.json` | Worked example, scores 100/100 (50 min, Grade 7) |
| `assets/examples/03-history-source-comparison.plan.json` | Worked example, scores 98/100 (45 min, Year 10, non-US framing) |
| `assets/examples/02-math-fractions-FLAWED.plan.json` | **Deliberately broken**, scores 24/100 with 9 hard failures |

Run the flawed example through the validator when you want to see what each gate looks like when it fires:

```bash
python scripts/validate_plan.py assets/examples/02-math-fractions-FLAWED.plan.json
```

## Teaching outside a US classroom

The arc, the ceiling, and the practice floor were derived from US classroom conventions. `references/context-adaptation.md` covers what to change: period length (40/45/50 rather than a default 50), standards alignment (national curriculum rather than state standards), and the substitute-teacher test (there may be no substitute role — treat it as the *handover test*: could a colleague pick this up cold?).

Four things do **not** adapt and should not be softened for local custom: the 15-minute direct ceiling, the 50% practice floor, the observable verb, and the numeric threshold with a named move. A curriculum framework phrased more vaguely does not make vagueness measurable.

## Deliverable

A single scannable document: objective, grade and duration, standards alignment if provided, the flow table with per-segment times, the formative check with its proceed/re-teach rule, the exit task, differentiation notes, and any assumptions marked as assumptions.

## Do NOT

- Do not start from a favourite activity and reverse-engineer an objective; the objective comes first or the lesson drifts.
- Do not let direct instruction exceed 15 minutes. Past that point retention drops and practice time disappears.
- Do not write a formative check without a numeric rule; "check for understanding" produces no decision.
- Do not write an exit task that tests something other than the objective's verb — an objective about comparing, checked by a recall quiz, measures nothing.
- Do not plan a unit here. One lesson per plan; route multi-week sequencing to `curriculum-mapper`.

## Quality bar

- The objective uses an observable verb, one primary target, and names a condition.
- Practice is at least half the lesson; instruction is at or under 15 minutes.
- The formative check names both a numeric threshold and a re-teach move.
- The exit task mirrors the objective's wording.
- Every segment says what students do and what they produce — a colleague could run minute 12 without asking a question.

---

Adapted from `SkillMedev/skills` → `skills/lesson-plan-builder/SKILL.md` (MIT licence). The `references/`, `scripts/`, and `assets/` trees were added locally; the original skill was a single Markdown file.
