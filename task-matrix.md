---
title: Task matrix
description: One row per unit of work, and a done status that requires evidence attached to it.
---

# The task matrix

`docs/dev-tasks/tasks.csv` is the todo list for a whole piece of work: one row per user
story, with its acceptance criteria, what it traces to, its priority and its status. Browse
it in a spreadsheet; drive it from the CLI or the MCP tools so the columns survive.

## Columns

| Column | Written by | Holds |
|--------|-----------|-------|
| `id` | you | `US-36`, `BUG-12` — what every other document is named after |
| `user_story` | you | As a &lt;role&gt;, I want &lt;capability&gt; so that &lt;outcome&gt; |
| `acceptance_criteria` | you | The observable behaviour, stated so a check can disagree with it |
| `traces_to` | you | The business rule or NFR this exists for |
| `priority` | you | HIGH / MEDIUM / LOW |
| `status` | the loop | `not-done` → `ready` → `in-progress` → `done`, or `blocked` |
| `testing_plan` | the loop | Set when the contract is agreed |
| `plan` | the loop | Set when the plan is approved |
| `validation` | the loop | Set when it passes |
| `notes`, `updated` | either | Why it is blocked; the date of the last change |

## Statuses

<pre class="mermaid">
stateDiagram-v2
    [*] --> not_done: imported
    not_done: not-done
    ready: ready
    in_progress: in-progress
    blocked: blocked
    done: done
    not_done --> ready: testing plan agreed
    ready --> in_progress: plan approved
    in_progress --> done: validation doc PASSES
    in_progress --> blocked: waiting on something outside the repo
    blocked --> in_progress: unblocked
    in_progress --> ready: contract changed
    done --> [*]
</pre>

`ready` means the definition of done exists and a human agreed to it. It does not mean work
has started.

## `done` is gated on evidence

A row reaches `done` only when a file in `docs/validation/` carries `**Task:** <id>` and
`**Verdict:** PASS`. The gate lives in one function, so the human CLI and an agent's MCP
tools hit the same refusal — a gate with two entrances is not a gate.

```markdown
# JEV-09 — validation

**Task:** JEV-09
**Method:** unit test
**Verdict:** PASS
**Date:** 2026-10-31

## What was checked

| Criterion (from the matrix) | How it was checked | Result |
|---|---|---|
| A typed error names the cause | the project's own test command | PASS |
| No probability is fabricated | same suite, `ProviderFailures` | PASS |
```

Pick whatever method the project already uses:

| The criterion is about | Method |
|------------------------|--------|
| A pure function, a branch, an error path | `unit test` |
| Several components together, a migration, a job | `integration test` |
| An endpoint, a payload contract, an auth rule | `newman` / postman |
| Structure, naming, a security or design rule | `code review` |
| Something genuinely unautomatable | `manual`, with the steps written out |

**A criterion nothing checks is not a PASS.** It belongs under "Not covered", which makes
the verdict PARTIAL and honestly keeps the row open.

## The operations

Deliberately few. Read the matrix; ask what to work on next; read one row with its criteria
verbatim; move a row; ask what evidence exists for a row; add a row; import rows from a
pasted user-story table. Moving a row to `done` is the only one that can be refused.

A forced override exists for a human who explicitly asks, and it records that fact in the
row. A hole you can see in the chart beats one people work around by editing the record by
hand — which is the real failure mode, and the reason the override is written down rather
than removed.

## What this gate cannot check

That the evidence is about *this* row. The paperwork can be in order while the work is
about something else entirely. Closing that gap needs a judgement rather than a rule, and
that is [the second gate]({{ site.baseurl }}/second-gate.html) — a design, not a feature.
