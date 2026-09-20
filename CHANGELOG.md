# Changelog

All notable changes to this skill are documented here.
This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] — 2026-09-20

First release. Turns a single-file prompt into a working toolkit with an
executable quality gate.

### Added

**`references/` — 9 lookup documents** (the skill previously had none)

- `blooms-verbs.md` — observable verb bank by Bloom level, plus explicit banned
  and "weak but allowed" lists
- `objective-patterns.md` — canonical objective form, rewrite drills, conditions
  worth naming, anti-patterns
- `exit-task-patterns.md` — the mirroring rule, smallest faithful task form per
  verb family
- `activity-library.md` — activities indexed by evidence generated and by arc
  phase, plus the ones that reliably fail the objective filter
- `formative-check-design.md` — the three required parts, placement, why
  procedural lessons want two checks
- `differentiation-scaffolds.md` — scaffolds ordered strongest to weakest;
  extensions that stay on the objective
- `time-budget-table.md` — derived budget for 20–90 minutes, two-cycle structure
  for double periods, common misallocations
- `quality-rubric.md` — the fourteen weighted checks and score interpretation
- `context-adaptation.md` — adapting the arc outside a US classroom

**`scripts/` — 4 stdlib-only tools** (Python 3.9+, no dependencies)

- `lesson_lib.py` — verb banks, budget derivation, Markdown parser, the
  fourteen-check rubric
- `budget.py` — duration → four-part time budget, single or ranged
- `build_plan.py` — plan JSON → lesson plan Markdown; segment times may be left
  `null` and are inherited from the budget
- `validate_plan.py` — plan (JSON or Markdown) → scored report with a gateable
  exit code
- `selftest.py` — 60 assertions over the whole chain, run by CI on 3.9/3.12/3.13

**`assets/` — templates and worked examples**

- `templates/plan.schema.json` — the input contract for `build_plan.py`
- `templates/lesson-plan.md` — output layout
- `templates/prep-checklist.md` — pre-lesson checklist, printer-first ordering
- Three worked examples: a 100/100 biology plan, a 98/100 45-minute history
  plan, and a deliberately broken 24/100 maths plan for calibration

**Repository**

- `README.md`, `LICENSE` (MIT), CI workflow

### Changed

- `SKILL.md` rewritten for progressive disclosure: the procedure now points into
  `references/` instead of inlining everything, and the command surface is
  documented against the scripts.
- `name` in frontmatter corrected from `Lesson Plan Builder` to
  `lesson-plan-builder` so it matches the directory name.

### Fixed

Three bugs found by running the toolkit rather than only writing it.

- **Validator ignored budget-inherited times.** A plan may leave a segment's
  `minutes` null and let the budget supply it. `validate_plan.py` judged the raw
  input, so every inherited time was reported as missing: the same plan scored
  **58/100 as JSON and 100/100 as Markdown**. Resolution now lives in
  `lesson_lib.resolve()` and both the renderer and the validator use it.
  (`build_plan.py --check` had the same defect.)
- **`understand` escaped detection.** An objective written
  `Students will understand …` — without the `will be able to` stem — fell
  through to a naive first-word fallback that read `Students` as the verb, so the
  most common banned verb in the wild produced only a soft warning. Verb
  extraction now handles the modal form and strips a leading subject.
  (`extract_objective_verb`)
- **The renderer was stricter than the validator.** An empty
  `differentiation.extension` aborted the build, which made the validator's
  "both support and extension required" check unreachable. Rendering now
  tolerates a blank field and leaves the judgement to `validate_plan.py`.

Also corrected a garbled pass-condition in the rubric table (check 6 read
"Otherwise not stated, or stated above 15") and filled in the two blank cells
for checks 8 and 9.

### Attribution

The original `skills/lesson-plan-builder/SKILL.md` came from the
[SkillMedev/skills](https://github.com/SkillMedev/skills) catalogue (MIT) and
supplied the 8-step procedure, the four-part arc, and the quality bar. Everything
above was added on top of it.
