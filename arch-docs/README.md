# docs/arch-docs/

The design a change has to fit into. `/eval` scans this folder for the section relevant to
the ticket, so the constraints written here become constraints in the plan.

One file per subject, not per ticket: `<subject>.md` — `request-flow.md`, `data-model.md`,
`auth.md`. A ticket-shaped file belongs in `docs/dev-tasks/`.

## What belongs here

- The shape of a flow: who calls what, in what order, with what payload.
- Data contracts: table keys, message headers, response envelopes.
- Decisions and the reason behind them, especially the ones a newcomer would undo.
- Boundaries: what this service owns and what it must not touch.

## What does not

- Step-by-step implementation plans (`.plans/`).
- Acceptance criteria (`docs/dev-tasks/`).
- How to run things locally (`AGENTS.md` or the project README).

## Suggested shape

```markdown
# <Subject>

**Last reviewed:** YYYY-MM-DD

## Overview

One paragraph a new dev can hold in their head.

## Flow

<sequence, numbered or as a diagram>

## Contracts

<tables, schemas, headers, status codes>

## Decisions

| Decision | Reason | Date |
|----------|--------|------|
| ... | ... | ... |

## Constraints

Rules a change in this area must not break.
```
