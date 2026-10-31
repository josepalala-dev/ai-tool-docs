---
title: Reference
description: The repository's own documents, the semantic-gate prototype, and how to contribute.
---

# Reference

The canonical documents live in the repository and are not duplicated here, so this site
cannot drift out of date with them.

## In the repository

| Document | What it covers |
|----------|----------------|
| [README](https://github.com/josepalala-dev/ai-sdlc-tool/blob/master/README.md) | Full reference: flags, skills, matrix, MCP, applying it elsewhere |
| [ELI5-MCP](https://github.com/josepalala-dev/ai-sdlc-tool/blob/master/ELI5-MCP.md) | Why MCP, from scratch, with the honest caveats |
| [CONTRIBUTING](https://github.com/josepalala-dev/ai-sdlc-tool/blob/master/CONTRIBUTING.md) | What is worth sending, what will be declined, the loop to follow |

## Design decisions

Written down so the reasons survive the people:

- [Task gates]({{ site.baseurl }}/arch-docs/task-gates.html) — why `done` is the only gated
  status, why the gate lives in one function, why minimum rather than mean, why a low score
  parks a row instead of failing a build.
- [Decision model]({{ site.baseurl }}/arch-docs/decision-model.html) — the provider seam,
  the configuration, the provenance and its limits.

## Document formats

- [Ticket format]({{ site.baseurl }}/dev-tasks/README.html) — one file per ticket
- [The matrix]({{ site.baseurl }}/dev-tasks/TASKS.html) — columns and statuses
- [Working a ticket]({{ site.baseurl }}/dev-tasks/WORKFLOW.html) — the loop, as scaffolded
- [Testing plan]({{ site.baseurl }}/testing-plan/README.html) — the contract for "done"
- [Validation]({{ site.baseurl }}/validation/README.html) — the evidence format
- [Arch docs]({{ site.baseurl }}/arch-docs/README.html) — what belongs in a design doc

## The semantic gate — prototype

The paperwork gate asks *is there evidence?* It cannot ask *is the evidence about **this**
row?* Closing that gap needs a judgement, not a regex: does this diff address these
acceptance criteria?

The design is a second gate behind the first, using a System One decision model
([TypeSafe's Jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev)) — one
probability per acceptance criterion, cheap and fast enough to run on every row.

```text
task_assess(id="JEV-09")
   in:  the row's acceptance criteria, verbatim
        the diff that claims to implement them
        the artifacts the validation doc names
   out: a probability per criterion, and the ones nothing addressed
```

**Status: prototype.** The provider seam and connection are implemented and tested offline;
the assessment layer and the tool itself are not finished. It is **off unless configured**,
and the repository stays standard-library-only.

Three limits stated plainly:

1. **Typed is not true.** The schema constrains the shape of the answer; it does not prove
   the judgement inside it. A calibrated 0.93 is a good bet, not a proof.
2. **Thresholds do not transfer.** What counts as passing is empirical per project and per
   model. A default is a starting point, not a calibration.
3. **It does not choose the criteria.** If the row says the wrong thing, a verdict that the
   build matches the row is a confident yes to the wrong question.

## Contributing

This is MIT licensed, so you may take it and never say a word. Please don't — the licence is
the floor, not the ask. A method only improves from contact with projects it was not
designed for.

Most useful of all: a report from real use. Which parts you kept, which your team silently
skipped, and what your thresholds settled at.

<div class="cta">
  <a class="primary" href="https://github.com/josepalala-dev/ai-sdlc-tool/issues/new?labels=field-report&title=%5Bfield-report%5D%20">File a field report</a>
  <a href="https://github.com/josepalala-dev/ai-sdlc-tool/issues/new?labels=bug&title=%5Bbug%5D%20">Report a bug</a>
  <a href="https://github.com/josepalala-dev/ai-sdlc-tool/issues">All issues</a>
</div>
