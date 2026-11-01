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
    R1["the ticket"] -.read.-> E
    R2["the project description"] -.read.-> E
    R3["the design notes"] -.read.-> E
    T -.writes.-> W1["the testing plan"]
    P -.writes.-> W2["the plan (scratch)"]
    D -.writes.-> W3["code"]
    V -.writes.-> W4["the validation record"]
</pre>

## 1 · `/eval` — read reality

Reads the files, pulls the acceptance criteria **verbatim**, says what exists and what is
missing, and names one real risk per file it will touch. No planning, no code.

It reads the ticket for the criteria, the project description for the standards, and the
design notes for the shape it has to fit.

## 2 · Testing plan — write the contract

One document per ticket. Every acceptance criterion becomes a row with a stated check.
**A criterion you cannot check is a criterion you cannot claim.**

Get this reviewed before step 3: it is cheaper to argue about the contract than the code.
Then the matrix row goes to `ready`, which means the definition of done exists — not that
work has started.

## 3 · `/write-plan` — the blueprint

Numbered steps, one file change each, with `[ R ]` / `[ IP ]` / `[ D ]` markers. Still no
code. Ends waiting for approval.

The plan is working scratch and is deliberately throwaway. Its durable outputs are the
testing plan and the matrix row.

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
| Branching, deleting, force-pushing, opening a PR | `/git-safety` — optional, and not a stage of the loop |

## Alongside the loop: git safety

Optional, and deliberately not a step. The loop grades whether the work is *right*;
`/git-safety` is about the operations around it that are *unrecoverable* if wrong.

It draws one hard line: **an agent never deletes anything on a remote.** Not a branch, not
a tag, not a release — not when the branch is merged, not when it is obviously stale, not
when the human said "clean it up". Remote deletion reaches other people's clones and open
reviews and is effectively irreversible, so the agent prints the command and the human runs
it themselves. Everything local is fair game once containment is actually verified, because
a local mistake costs one person a few minutes.

The same test applies to rewriting published history: allowed when asked, but never a blind
force-push, and the old and new commit ids get reported so anyone holding the old one can
recover.

### Release safety

A second optional part, for projects that have a release process at all.

The default branch is a guess. Plenty of projects integrate somewhere else — a development
branch, a staging branch, a long-lived release branch per train, or a per-ticket
integration branch — and branching off the wrong base or opening a review against the wrong
target is not destructive, just expensive: a rebase, a re-review, sometimes a reverted
merge.

So before starting a branch or opening a review, read the project's own notes for the real
target, and **quote the line that named it.** That is the difference between an auditable
choice and a hunch. When nothing names it and more than one target is plausible, ask — one
question is cheaper than a review retargeted afterwards. Two traps worth naming:

- **Never infer the target from a merge-commit message.** Ticket ids get reused across
  release branches, so a matching commit subject proves nothing about where the work landed.
  Prove containment with the tooling instead.
- **A branch far behind its target** will show a review full of other people's work and
  conflict at merge time. Report how far behind it is rather than opening the review on top
  of that.

Documented deployment steps get the same treatment as remote deletion: follow the project's
flow rather than a general-purpose guess, and any step the notes describe as a human's —
cutting a tag, approving a promotion, touching production configuration — stays with the
human.

Both parts follow the question the rest of the loop never asks: *if this is wrong, who has
to clean it up?* When the answer is somebody else, the agent hands it over instead of being
helpful.

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
