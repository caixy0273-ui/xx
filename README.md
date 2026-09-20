# lesson-plan-builder

An agent skill that builds **one complete, teachable lesson plan** — a measurable objective, a timed four-part arc, a formative check with a numeric decision rule, and differentiation notes — clear enough that a colleague could run it without asking a question.

It ships an **executable quality gate**: the skill's quality bar is implemented as a scoring validator, not just described in prose. Score a plan, get a number, get the failures named and weighted.

```
$ python scripts/validate_plan.py assets/examples/01-biology-photosynthesis.plan.json
Score 100/100 - ready to teach
```

```
$ python scripts/validate_plan.py assets/examples/02-math-fractions-FLAWED.plan.json
FAIL [10] Objective uses an observable verb: 'understand' cannot be observed.
FAIL [12] Exit task mirrors the objective's verb: Objective verb is 'understand' but
          the exit task never uses it.
FAIL [12] Direct instruction <= 15 min: Direct instruction is 30 min, over the ceiling.
FAIL [12] Practice >= 50% of the lesson: Practice is 13 min = 26%.
...
24/100 - rebuild from the objective
```

## The failure this prevents

The lecture-heavy plan. The teacher talks for 30 minutes, practice gets squeezed into the last 5, and the first real evidence of confusion arrives on the graded test — when it is too late to act.

Every rule in this skill exists to make that plan impossible to write by accident.

## What's in it

| Path | Contents |
|---|---|
| `SKILL.md` | The entry point: 8-step procedure, the four quality gates, script usage, routing |
| `references/` | 9 lookup documents — verb bank, objective patterns, exit-task forms, activity library, formative-check design, scaffolds, time-budget table, the 14-check rubric, non-US classroom adaptation |
| `scripts/` | 4 stdlib-only Python tools — budget, build, validate, shared logic |
| `assets/` | Input schema, output template, prep checklist, and 3 worked examples |

**Standard library only, Python 3.9+.** No install step, no dependencies.

## Quick start

```bash
# 1. What does a 45-minute lesson actually look like?
python scripts/budget.py --duration 45

# 2. Build a plan from JSON (leave times null to inherit the budget)
python scripts/build_plan.py --in my-plan.json --out my-plan.md --check

# 3. Validate any plan — JSON or Markdown, hand-written or generated
python scripts/validate_plan.py my-plan.md
python scripts/validate_plan.py my-plan.json --json
python scripts/validate_plan.py my-plan.md --min-score 90
```

`validate_plan.py` exits non-zero when any check hard-fails or the score is below `--min-score` (default 75), so it can gate a generation loop.

Try it on the worked examples:

```bash
python scripts/build_plan.py --in assets/examples/01-biology-photosynthesis.plan.json --check
python scripts/validate_plan.py assets/examples/03-history-source-comparison.plan.json
```

## The four gates

The method is backward design — **objective → exit task → activities**, in that order. Choose the activity first and the activity smuggles in its own objective.

| Gate | Rule | What it blocks |
|---|---|---|
| **Objective** | Observable verbs only (`identify`, `compare`, `construct`, `evaluate`) + a named condition | `understand`, `appreciate` — nothing to observe, no finish line |
| **Exit task** | Must mirror the objective's verb | An objective about *comparing*, checked by a *recall* quiz, measures nothing |
| **Time** | Direct instruction ≤ 15 min **hard ceiling**; practice ≥ 50% of the lesson | Content that doesn't fit becomes two lessons, not a longer lecture |
| **Check** | Needs a visible whole-class instrument, a **numeric** threshold, and a **named** re-teach move | "Check for understanding" with no threshold produces no decision |

Then the handover test: read the plan as someone who has never met the class. Every segment must say what students do *and* what they produce.

## Worked examples

| File | Score | Why it's here |
|---|---|---|
| `01-biology-photosynthesis.plan.json` | **100/100** | 50 min, Grade 7 — the target shape |
| `03-history-source-comparison.plan.json` | **98/100** | 45 min, Year 10, non-US framing — shows the arc at a different period length |
| `02-math-fractions-FLAWED.plan.json` | **24/100** (9 hard failures) | **Deliberately broken** — run it to see what each gate looks like when it fires |

The flawed example is the useful one. It is a realistic bad plan: a banned verb, a 30-minute lecture, 26% practice, a threshold with no number, and a re-teach move that says "review it".

## Scope boundaries

- **Does** build one lesson plan, start to finish.
- **Does not** sequence a multi-week unit — use `curriculum-mapper`.
- **Does not** adapt an existing lesson per learner — use `differentiated-instruction`.
- **Does not** write the quiz — use `quiz-generator`.

## Teaching outside a US classroom

The arc, the ceiling, and the practice floor derive from US classroom conventions. `references/context-adaptation.md` covers what to change: period length (40/45/50 rather than a default 50), standards alignment (national curriculum rather than state standards), and the substitute-teacher test (where there is no substitute role, read it as a *handover test*).

Four things do **not** adapt: the 15-minute direct ceiling, the 50% practice floor, the observable verb, and the numeric threshold with a named move. A curriculum framework phrased more vaguely does not make vagueness measurable.

---

## 中文说明

这是一个**备课技能**：把"明天这节课怎么上"变成一份可以直接执行的单课时教案。

它防的是那个代价很高的失败——老师连讲 30 分钟，练习被挤到最后 5 分钟，**第一次发现学生没听懂，是在批改考卷的时候**，那时已经来不及补救了。

方法是**逆向设计**：先定可观测的目标，再写出题，最后才选活动。因为"先选活动"会让活动偷偷把自己的目标塞进来。

四道质量门：目标只用可观测动词（拒绝 `understand`）／出题必须镜像目标的动词／讲授硬顶 15 分钟且练习不少于一半课时／检查点必须同时给出数字阈值和具体的重教动作。

`scripts/validate_plan.py` 把上述规则做成了**可执行的校验器**，输出 100 分制评分与加权失败项，可用于门禁。

标准库实现，Python 3.9+，无需安装任何依赖。

## Licence

MIT. See `LICENSE`.

The original `skills/lesson-plan-builder/SKILL.md` came from the [SkillMedev/skills](https://github.com/SkillMedev/skills) catalogue (MIT). The `references/`, `scripts/`, and `assets/` trees and the rewritten `SKILL.md` were added on top of it. Attribution is retained in `LICENSE` and at the foot of `SKILL.md`.
