# Task gates

**Last reviewed:** 2026-10-31

## Overview

A matrix row cannot be marked `done` on anyone's word. It has to get past gates, and there
are two of them: the **paperwork gate**, which is in and enforced today, and the
**semantic gate**, which is prototype and optional. The first asks "is there evidence?" The
second asks "is the evidence about *this* row?"

Both live behind one function — `task_master.set_status` — because a gate with two entrances
is not a gate. The CLI and the MCP tools both call it, so a human and an agent hit the same
refusal.

## Flow

```
set_status(id, status)
   │
   ├── status != "done"  ──────────────────────────────▶ write the row, stop
   │        (ready / in-progress / blocked / not-done never consult a model)
   │
   ├── force=True  ────────────────────────────────────▶ write the row, record that it was forced
   │
   └── status == "done"
          │
          ├─ 1. paperwork gate — validation_gate(project, id)
          │     scans docs/validation/*.md for **Task:** <id>
          │     requires **Verdict:** PASS and a **Method:** line
          │     no doc, or no PASS  ──▶ MatrixError, row unchanged
          │
          ├─ 2. semantic gate — assessor(project, row), only if one is configured
          │     one Noul question per acceptance criterion
          │     every criterion must clear the threshold
          │     below threshold  ──▶ MatrixError naming the criterion, row unchanged
          │     provider error    ──▶ DecisionModelError propagates, row unchanged
          │
          └─ both passed ──▶ status=done, validation + assessment recorded, updated stamped
```

## Contracts

**Validation document** — `docs/validation/<id>-<slug>.md`. Four parsed header fields; the
rest is for humans:

| Field | Required | Parsed as |
|-------|----------|-----------|
| `**Task:**` | yes | whitespace/comma-separated ids; the doc claims only these |
| `**Method:**` | yes for a PASS | free text; `unit test`, `integration test`, `code review`, `newman`, `manual` |
| `**Verdict:**` | yes | `PASS` \| `FAIL` \| `PARTIAL`, upper-cased on read |
| `**Date:**` | no | free text |

A doc that merely mentions an id in its body does not count as that row's evidence. Only the
`**Task:**` header claims a row.

**Assessment** — recorded on the row as one auditable line carrying provider, model,
threshold and the minimum score. The matrix stores the summary, never the raw provider
response.

**Matrix columns the gates write:** `status`, `validation`, `assessment`, `notes`, `updated`.

## Decisions

| Decision | Reason | Date |
|----------|--------|------|
| `done` is the only gated status | The other four are claims about intent, not about evidence. Gating them would make the tool annoying without making it honest | 2026-10-31 |
| The gate lives in `set_status`, not in the CLI or the MCP layer | An implementer that can route around the gate will. One chokepoint means the human and the agent get the same refusal | 2026-10-31 |
| Evidence is a file on disk, not a field in the CSV | A path can be reviewed, diffed and argued with six months later. A boolean cannot | 2026-10-31 |
| Only the `**Task:**` header claims a row | Otherwise a doc that cites a neighbouring row in passing becomes that row's evidence by accident | 2026-10-31 |
| A PASS also requires a `**Method:**` | "It passed" without a named check is the exact claim the gate exists to refuse | 2026-10-31 |
| `--force` exists, and writes into `notes` | A hole you can see in the chart is better than one people work around by hand-editing the CSV | 2026-10-31 |
| One question per acceptance criterion, not one per row | A single blended score cannot say *which* criterion nothing addressed, and that sentence is the whole value of the check | 2026-10-31 |
| Every criterion must clear the threshold (min, not mean) | Averaging lets two well-covered criteria carry an uncovered third | 2026-10-31 |
| A low assessment parks the row (`blocked` + notes) rather than failing the run | An uncertain verdict is a hand-off to a human, not a broken build | 2026-10-31 |
| A provider error leaves the row untouched and propagates | An unreachable model is not a judgement. Treating it as a FAIL would teach people to disable the gate | 2026-10-31 |
| The semantic gate is off unless configured | This is a standard-library tool. Most users want the paperwork gate and nothing else | 2026-10-31 |
| The paperwork gate stays authoritative | A high score never substitutes for a missing validation doc, or a model becomes the thing that closes rows | 2026-10-31 |

## Constraints

- `set_status` is the chokepoint for all five statuses. A change here breaks every status
  transition, not just `done`.
- No status change may invent evidence. If a check cannot run, the row keeps its old status.
- The paperwork gate must keep working with no provider, no SDK and no network.
- Nothing in the semantic path may be imported at `task_master` load time.
- The matrix never stores a raw provider response — only a summary line.
