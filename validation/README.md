# docs/validation/

One file per validated task, named `<task-id>-<slug>.md`. **A row in the task matrix cannot
be marked `done` until a file in here names it and records a PASS.** `task_master.py`
enforces that; it is not a convention anyone has to remember.

Why the gate exists: an implementer that can tick off its own work ticks it off. The matrix
tracks evidence instead, and this folder is the evidence.

## Format

```markdown
# <TASK-ID> — validation

**Task:** <TASK-ID>
**Method:** unit test | integration test | code review | newman | manual
**Verdict:** PASS | FAIL | PARTIAL
**Date:** YYYY-MM-DD
**Testing plan:** `docs/testing-plan/<task-id>-<slug>.md`

## What was checked

| Criterion (from the matrix) | How it was checked | Result |
|-----------------------------|--------------------|--------|
| <acceptance criterion, verbatim> | `pytest tests/test_x.py::test_y` | PASS |
| <criterion> | newman run `postman/tickets.json` folder "Email intake" | PASS |
| <criterion> | code review — see findings below | PARTIAL — blocked on SMTP creds |

## Evidence

```
<the command that was run, and enough of its output to be checkable>
```

## Findings

- <anything a reviewer raised, and what happened to it>

## Not covered

- <criterion nobody could check, and why — this is what makes the verdict PARTIAL>
```

The four `**Task:**`, `**Method:**`, `**Verdict:**` and `**Date:**` lines are parsed, so keep
their wording exactly. Everything below them is for humans.

## Picking a method

Whatever the project already uses — the gate is about having evidence, not about a
particular test runner.

| The criterion is about | Method |
|------------------------|--------|
| A pure function, a branch, an error path | unit test |
| Several components together, a migration, a job | integration test |
| An HTTP endpoint, a payload contract, an auth rule | newman / postman — see `/api-testing` |
| Structure, naming, a security or design rule | code review |
| Something genuinely unautomatable (an email landing in an inbox) | manual, with the steps written out |

A criterion checked by nothing is not a PASS. It belongs under **Not covered** and makes the
verdict PARTIAL, which keeps the row out of `done` honestly rather than quietly.

## Verdicts

- **PASS** — every criterion in the matrix row was checked and held. The only verdict that
  lets the row be marked done.
- **PARTIAL** — something is blocked on an external thing. Name it. The row stays
  `in-progress` or goes `blocked`.
- **FAIL** — a criterion did not hold. Back to `/eval`.
