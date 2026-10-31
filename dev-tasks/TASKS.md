# The task matrix

`tasks.csv` (or `tasks-<randomhex>.csv` when a project runs more than one) is the todo list
for a whole piece of work: one row per unit, checked off as the loop runs. Browse it in any
spreadsheet; drive it from `task_master.py` or the `task-master` MCP tools so the columns
survive.

## Columns

| Column | Written by | What it holds |
|--------|-----------|---------------|
| `id` | you | `US-36`, `BUG-12` — what every other document names in its filename |
| `user_story` | you | As a &lt;role&gt;, I want &lt;capability&gt; so that &lt;outcome&gt; |
| `acceptance_criteria` | you | The observable behaviour, stated so a check can disagree with it |
| `traces_to` | you | The business rule or NFR this exists for — `BR-N01, NFR-04` |
| `priority` | you | HIGH / MEDIUM / LOW |
| `status` | the loop | `not-done` → `ready` → `in-progress` → `done`, or `blocked` |
| `testing_plan` | the loop | `docs/testing-plan/<id>-<slug>.md`, set when the contract is agreed |
| `plan` | the loop | `.plans/<date>-<slug>.md`, set when the plan is approved |
| `validation` | the loop | `docs/validation/<id>-<slug>.md`, set when it passes |
| `notes` | either | Why a row is blocked, or what was cut |
| `updated` | the loop | Date of the last status change |

## Statuses

| Status | Means | Set when |
|--------|-------|----------|
| `not-done` | In the matrix, nothing written yet | `init` |
| `ready` | The contract exists and a human agreed to it | The testing plan is reviewed |
| `in-progress` | A plan is approved and being applied | `/do-it` starts |
| `blocked` | Waiting on something outside the repo — `notes` says what | Any time |
| `done` | A validation doc names it and records a PASS | `/verify-it` passes |

`done` is gated. `task_master.py set <id> done` is refused unless a file in
`docs/validation/` carries `**Task:** <id>` and `**Verdict:** PASS` — see
`docs/validation/README.md`. That is deliberate: an unforced `done` always has evidence
behind it.

## Seeding it

Paste a markdown matrix into a file and seed from it — the first five columns are read
positionally, in the order above:

```bash
python3 task_master.py init . --from backlog.md
```

Or write the rows straight into the CSV. Either way the acceptance criteria column is the
one that matters: `/eval` quotes it, the testing plan turns it into rows, and the validation
doc checks it off one by one.
