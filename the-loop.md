---
title: The loop
description: Five steps from a matrix row to a closed task, and what each one reads and writes.
---

# The loop

Five steps against one row of the task matrix. Each has a slash command except the testing
plan, which is yours to write.

<pre class="mermaid">
flowchart LR
    E["/eval"] --> T["testing plan"]
    T --> P["/write-plan"]
    P --> D["/do-it"]
    D --> V["/verify-it"]
    R1["docs/dev-tasks/"] -.read.-> E
    R2["AGENTS.md"] -.read.-> E
    R3["docs/arch-docs/"] -.read.-> E
    T -.writes.-> W1["docs/testing-plan/"]
    P -.writes.-> W2[".plans/ (scratch)"]
    D -.writes.-> W3["code"]
    V -.writes.-> W4["docs/validation/"]
</pre>

## 1 · `/eval` — read reality

Reads the files, pulls the acceptance criteria **verbatim**, says what exists and what is
missing, and names one real risk per file it will touch. No planning, no code.

It looks in `docs/dev-tasks/` for the ticket, `AGENTS.md` for standards, and
`docs/arch-docs/` for the design it has to fit.

## 2 · Testing plan — write the contract

`docs/testing-plan/<id>-<slug>.md`. Every acceptance criterion becomes a row with a stated
check. **A criterion you cannot check is a criterion you cannot claim.**

Get this reviewed before step 3: it is cheaper to argue about the contract than the code.
Then the matrix row goes to `ready`, which means the definition of done exists — not that
work has started.

## 3 · `/write-plan` — the blueprint

Numbered steps, one file change each, with `[ R ]` / `[ IP ]` / `[ D ]` markers. Still no
code. Ends waiting for approval.

`.plans/` is working scratch and is not committed. Its durable outputs are the testing plan
and the matrix row.

## 4 · `/do-it` — implement

Follows the approved plan in order, flipping markers as it goes. Anything the plan did not
account for **stops the run** instead of getting improvised.

Use `/execute-plan` when the plan is long: same work in batches with a review checkpoint
after each, so a wrong assumption costs a batch instead of the whole run.

## 5 · `/verify-it` — prove it, then tick the row off

Runs the gates (lint, typecheck, test, build), then walks the testing plan row by row:
PASS / FAIL / PARTIAL, where PARTIAL means blocked on something external and names it.

Then write `docs/validation/<id>-<slug>.md` and close the row. That last step is
[gated]({{ site.baseurl }}/task-matrix.html): without a passing validation document, the
status change is refused.

## Starting earlier, or elsewhere

| Situation | Where to start |
|-----------|----------------|
| The ask is still vague | `/brainstorming` — explore requirements and design, write them down, stop |
| A one-file fix | Step 3. Nothing starts at step 4: with no plan there is nothing to execute |
| A bug | `/debugging` instead of the loop — investigate, find the pattern, test hypotheses, then fix. It escalates to architectural review after three failed attempts rather than trying a fourth |
| HTTP endpoints | `/api-testing` for the rows unit tests cannot cover |

## Roles, and how each one fails

Work is split by role because each role has a different failure mode, and an implementer
that grades its own work grades generously.

| Role | Reads | Writes | Fails by |
|------|-------|--------|----------|
| Explorer | the ask, the codebase | requirements, design | inventing requirements nobody asked for |
| Planner | requirements, design, contract | the task list | writing code in the plan |
| Implementer | the task list | code | exceeding the plan |
| Reviewer | diff + plan + `AGENTS.md` | findings | reviewing taste instead of contract |
| Verifier | the contract | pass/fail | claiming PASS without running it |

Run them as separate agents where the tooling allows, separate turns where it doesn't.
Parallel agents are for independent work only — two on one file is a merge conflict with
extra steps.
