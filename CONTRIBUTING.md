---
title: Contributing
description: The source is private and the method is public. Here is what that means for contributing, and why reports are worth more than patches here.
---

# Contributing

**The source is private. This repository — the documentation — is public, and it is where
contribution happens.**

That is an unusual shape, so it is worth being straight about what it means before asking
anything of you.

## What is public and what is not

| | Where | Licence |
|---|---|---|
| The method — the loop, the matrix, the gate, the document formats | This repository, and the [site](https://josepalala-dev.github.io/ai-tool-docs) | MIT |
| The implementation — the tooling that enforces it | Private | not published |

The method is the part worth having, and the method is the part that is published. What
makes this work is not the tooling — it is writing the contract for "done" before the code
exists, and refusing to mark anything done without evidence attached. Those are habits, and
habits do not need anyone's source code.

So the usual open-source ask — *send a pull request* — only partly applies here. Against
the documentation it applies fully. Against the implementation it cannot, and pretending
otherwise would waste your time.

## Where to file

Everything goes to this repository.

| What you have | Where it goes |
|---|---|
| A report from real use — thresholds, a false refusal, a step your team skipped | [New field report](https://github.com/josepalala-dev/ai-tool-docs/issues/new?labels=field-report&title=%5Bfield-report%5D%20) |
| A validation method the formats don't cover | [New validation method](https://github.com/josepalala-dev/ai-tool-docs/issues/new?labels=validation-method&title=%5Bvalidation%5D%20) |
| Something in the docs that is wrong, stale, or unclear | [New docs issue](https://github.com/josepalala-dev/ai-tool-docs/issues/new?labels=docs&title=%5Bdocs%5D%20) — or just open a pull request |
| A bug in the behaviour the docs describe | [New bug report](https://github.com/josepalala-dev/ai-tool-docs/issues/new?labels=bug&title=%5Bbug%5D%20) |
| An idea, or a question about whether something is wanted | [Open an issue](https://github.com/josepalala-dev/ai-tool-docs/issues/new) |

A look at [open](https://github.com/josepalala-dev/ai-tool-docs/issues) and
[closed](https://github.com/josepalala-dev/ai-tool-docs/issues?q=is%3Aissue+is%3Aclosed)
issues first saves a duplicate — a closed issue often carries the reasoning for a decision
that is written down nowhere else.

Bug reports are still worth filing even though you cannot see the code. Describe it against
what the documentation claims: the matrix row, the command, what the docs say should happen
and what happened. A behaviour that contradicts its own documentation is a bug in one of
them, and which one is the maintainer's problem, not yours.

## Pull requests, against the docs

This repository takes them normally. Fork, branch, open a PR. It is plain markdown served
by GitHub Pages — no build step, no npm, no theme gem — so what you see in the diff is what
ships.

Especially wanted:

- Corrections to `arch-docs/`, where a decision's stated reason is wrong or has gone stale.
  A reason that no longer holds is worse than no reason, because it stops people
  re-examining the decision.
- A validation format the existing ones don't cover, as an addition to
  `validation/README.md`.
- Places where the docs describe an intention rather than the behaviour. The Jev gate is a
  **prototype**; if a page reads as though it ships, that is a documentation bug.

## Asking for a change in behaviour

If you want the process to work differently rather than the prose to read better, describe
it the way the process itself would:

1. **As a user story with acceptance criteria.** That is the format the change will be
   built against anyway, and it is enough to work from without either of us looking at
   code together.
2. **With the case for it.** What you were doing, what the process made you do instead,
   and what it cost. A description of the friction is more useful than a proposed fix,
   because the fix is the easy part.

You will be credited in the change that results.

## Why reporting back matters here specifically

This is not a library you call. It is a **method**, and a method only improves from contact
with projects it was not designed for. Every real use teaches something the author could
not have known:

- Which validation methods your stack actually needed — a newman run, a snapshot test, a
  migration check nobody here anticipated.
- Where the gate got in the way, and whether that was the gate working or the gate being
  wrong. A false refusal teaches people to reach for `--force`, which kills the gate
  entirely.
- Which thresholds held up once a real model was pointed at real diffs, since
  **calibration does not transfer** between projects. This is the one thing no amount of
  design settles.
- Which parts of the loop your team silently skipped. That is the most valuable report of
  all, and the hardest to get.

Equally welcome: telling us a part of this is not worth its weight. A documented step that
changes nobody's behaviour should be deleted, and the author is the last person likely to
notice.

## Reporting without a patch

A report is a contribution — arguably the only kind that scales here. No template needs
filling in; prose is fine:

- "We used this for N weeks and dropped it after the planning step" — and why.
- "The validation doc format needed one more field for us."
- "Our threshold settled at X on project type Y."
- "We reimplemented the matrix in Go because …"

[Open a field report.](https://github.com/josepalala-dev/ai-tool-docs/issues/new?labels=field-report&title=%5Bfield-report%5D%20)
Say what you were building, what you kept and what you dropped.

## What will probably be declined

- **Anything that adds a dependency or a service.** The whole proposition is that this is
  cheap enough to adopt on a Tuesday. A lifecycle you have to install and maintain is one
  nobody runs.
- **Stack-specific rules.** The method knows nothing about any particular language,
  framework or team, and should stay that way. House rules belong in your own copy, not in
  the shared one.
- **Anything that weakens the gate.** Making `done` easier to reach defeats the only
  non-obvious thing this does. A better *check* is very welcome; a looser one is not.

## Security

Do not file a vulnerability as a public issue. Use
[private vulnerability reporting](https://github.com/josepalala-dev/ai-tool-docs/security/advisories/new),
which keeps the report between you and the maintainer until there is a fix. The
implementation is not published, so the useful report describes the behaviour rather than a
line number.

## Credit

Contributors keep their own copyright and are credited in the commit history, which is why
history here is not squashed. If a report or a patch shapes a decision, it gets named in
the relevant `arch-docs/` page alongside the reason — including when the contribution was
re-derived rather than merged.

The [second gate]({{ site.baseurl }}/second-gate.html) is itself modelled on
[foreman](https://github.com/thruwire/foreman) (MIT) — which is exactly the behaviour this
document is asking for: use the prior work, say that you did, and send back what you
learned.
