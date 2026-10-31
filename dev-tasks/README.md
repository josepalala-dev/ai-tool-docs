# docs/dev-tasks/

One file per ticket, named `<ticket-id>-<slug>.md`. This is what `/eval` reads first, so
anything missing here is context the loop has to guess at.

Keep it to what a dev needs to start. Design goes in `docs/arch-docs/`, the definition of
done goes in `docs/testing-plan/`.

## Format

```markdown
# <TICKET-ID> — <title>

**Status:** Not started | In progress | In review | Done
**Owner:** <name>
**Branch:** <branch name>

## User story

As a <role>, I want <capability>, so that <outcome>.

## Description

What is being changed and why. Include the trigger: a bug report, a compliance ask, a
dependency moving underneath us.

## Acceptance criteria

- [ ] <observable behaviour, stated so it can be checked>
- [ ] <one per line, no compound criteria>

## Dependencies

- <ticket, service, branch, or credential that must exist first>

## Out of scope

- <the thing a reviewer will otherwise ask about>

## Links

- Ticket: <url>
- Testing plan: `docs/testing-plan/<ticket-id>-<slug>.md`
- Plan: `.plans/<date>-<slug>.md`
```

## Writing acceptance criteria

A criterion is only useful if a check can disagree with it.

- Good: "A request with an unknown account id returns 404 and logs the id."
- Bad: "Lookups work correctly."

If you cannot name the check, the criterion is not ready and the testing plan will inherit
the problem.
