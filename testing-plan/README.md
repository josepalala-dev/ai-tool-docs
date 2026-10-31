# docs/testing-plan/

**Write the file in here before writing any code.** One per ticket, named
`<ticket-id>-<slug>.md`. This folder is the modified-TDD half of the loop: the contract
for "done" is agreed before the implementation exists, and `/verify-it` grades against it.

A testing plan is not a list of tests. It is the set of behaviours that must hold, each
with a named way to check it. Tests come later and may not cover every row — a row checked
by hand is fine as long as the plan says so.

## Format

```markdown
# Testing plan — <TICKET-ID> <title>

**Written:** YYYY-MM-DD (before implementation)
**Ticket:** `docs/dev-tasks/<ticket-id>-<slug>.md`

## Scope

What this change can break. Name the modules, endpoints, tables, topics, screens.

## Behaviours

| # | Behaviour that must hold | How it is checked | Type |
|---|--------------------------|-------------------|------|
| 1 | <observable outcome> | <test file, command, or manual steps> | unit / integration / e2e / manual |
| 2 | ... | ... | ... |

## Fixtures and setup

What has to exist for the checks to run: seeded rows, a token, a running dependency, an
env var. Say where each comes from.

## Negative cases

| # | Input or state | Expected handling |
|---|----------------|-------------------|
| 1 | <bad input, missing field, expired token> | <status, error shape, log line> |

## Regression watch

What already works that this change could quietly break, and the check that would catch it.

## Not covered

| What | Why | Risk accepted by |
|------|-----|------------------|
| <behaviour> | <needs prod data / external system> | <name> |
```

## Rules

1. Every acceptance criterion from the ticket appears as at least one row.
2. Every row names a check. "Verify it works" is not a check.
3. A behaviour that cannot be checked here goes in **Not covered** with a named owner —
   it never silently disappears.
4. The plan is reviewed before `/write-plan`. Changing the contract after the code exists
   means updating this file deliberately, not editing it to match what got built.
