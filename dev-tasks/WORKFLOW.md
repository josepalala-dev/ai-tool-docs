# Working a ticket in ai-sdlc-tool

The loop is five steps, run against one row of the task matrix. Each step has a slash
command except the testing plan, which is yours to write. **No code is written before the
testing plan exists, and no row is marked done without a validation document.**

```
/task-master  ->  /eval  ->  testing plan  ->  /write-plan  ->  /do-it  ->  /verify-it
 pick a row       read       the contract     the blueprint    implement    validate +
 from the CSV     reality    for "done"       (no code yet)                 tick the row off
```

The matrix — `docs/dev-tasks/tasks.csv` — is the todo list: one row per user story, with
its acceptance criteria, what it traces to, its priority and its status. Nobody tracks a
ticket's progress by hand; the row records which document each step produced. See
`docs/dev-tasks/TASKS.md`.

Start further back when the ask is still vague: `/brainstorming` explores requirements and
design and writes them to `docs/dev-tasks/` and `docs/arch-docs/` before step 1 exists.
`/ai-sdlc` is the map of the whole thing if you are unsure which step you are on.

## The rule

Writing code means writing a test plan first. Not tests — a *plan*. A testing plan names
the behaviours that must hold and how each one gets checked, before anyone knows what the
implementation looks like. That document is the definition of done. `/verify-it` grades
against it, so a vague testing plan buys you a vague verification.

Classic TDD writes one failing test and makes it pass. This loop does the same thing one
level up: write the whole contract down, get a human to agree with it, then build.

## 1. `/eval` — read reality

Reads the files, pulls the acceptance criteria verbatim, says what exists and what is
missing. Looks in `docs/dev-tasks/` for the ticket, `AGENTS.md` for standards, and
`docs/arch-docs/` for the design it has to fit. No planning, no code.

## 2. Testing plan — write the contract

Write `docs/testing-plan/<ticket-id>-<slug>.md` using the format in
`docs/testing-plan/README.md`. Every acceptance criterion from `/eval` becomes a row with
a stated check. A criterion you cannot check is a criterion you cannot claim.

Get this reviewed before step 3. It is cheaper to argue about the contract than the code.

## 3. `/write-plan` — the blueprint

Numbered steps, one file change each, written to `.plans/YYYY-MM-DD-<slug>.md` with
`[ R ]` / `[ IP ]` / `[ D ]` markers. Still no code. Ends waiting for your approval.

## 4. `/do-it` — implement

Or `/execute-plan` when the plan is long: same work, grouped into batches with a review
checkpoint after each one, so a wrong assumption costs a batch instead of the whole run.

Follows the approved plan in order, flips the markers as it goes, re-reads each file
before editing it. Anything the plan did not account for stops the run instead of getting
improvised.

## 5. `/verify-it` — prove it, then tick the row off

Runs the gates (lint, typecheck, test, build), then walks the testing plan row by row:
PASS / FAIL / PARTIAL, where PARTIAL means blocked on something external and names it.
Then a code review against the plan, `AGENTS.md` and the documented design. FAIL feeds back
into `/eval`.

Write the result to `docs/validation/<task-id>-<slug>.md` — the method (unit test,
integration test, newman/postman, code review, or a written-out manual procedure), the
verdict, and each acceptance criterion with the command that checked it. Then
`task-master set <id> done`, which is **refused unless that doc exists and says PASS**.
That refusal is the point: a `done` row always has evidence behind it, and a criterion
nothing checks is not a PASS.

For HTTP endpoints, the rows unit tests cannot cover belong to `/api-testing` — curl
scripts plus a collection that runs unattended, and a newman run is a perfectly good
validation method.

## Bugs

`/debugging` instead of the loop: investigate, find the pattern, test hypotheses, then fix.
It escalates to architectural review after three failed fix attempts rather than trying a
fourth.

## Where things live

| Path | What |
|------|------|
| `AGENTS.md` | Human-written description of this project: stack, commands, conventions |
| `docs/dev-tasks/tasks.csv` | The task matrix: one row per unit of work, and its status |
| `docs/dev-tasks/` | One file per ticket: user story, acceptance criteria, dependencies |
| `docs/testing-plan/` | The contract for "done", one per ticket, written before the code |
| `docs/arch-docs/` | Design the implementation has to fit |
| `docs/validation/` | The evidence a row is done: method, verdict, criteria checked off |
| `.plans/` | Implementation blueprints with step markers |

