---
title: The second gate
description: A design for measuring what was built against what the row asked for — probability per acceptance criterion. Prototype, and quite possibly not built yet.
---

# The second gate

{: .note }
**Status: a design, not a feature.** Treat everything below as how this would be built
rather than something you can switch on. Parts of it exist, parts of it are sketched, and
the parts that exist are off unless deliberately configured. If you are reading this
looking for a flag to set, there may not be one yet.

## The gap it closes

The [first gate]({{ site.baseurl }}/task-matrix.html) asks a question a regex can answer:
**is there evidence?** A row cannot reach `done` without a validation record naming it and
carrying a PASS.

That catches the common failure — ticking a box with nothing behind it. It does not catch
the subtler one:

> The evidence exists, it passes, and it is about something else.

A test suite goes green against criteria nobody checked it was testing. A validation record
names the right row and describes the wrong work. The paperwork is in order and the row is
still a lie. Nothing mechanical notices, because answering it needs a judgement: *does this
work address these criteria?*

## The shape of the answer

The wrong instinct is to ask a model "is this done?" — one question, one confident yes, and
the same self-certification problem one layer up.

The better shape is **one probability per acceptance criterion**:

```text
assess(row)
   in:  the row's acceptance criteria, verbatim
        the work that claims to implement them
        the artefacts the validation record names
   out: a probability for each criterion
        and, more usefully, the criteria nothing addressed
```

Four design choices make that useful rather than decorative:

**Per criterion, not per row.** A row with four criteria where three are nailed and one was
never touched is the normal case. A single verdict averages that into a shrug; four numbers
name the gap.

**Minimum, not mean.** The row is as good as its weakest criterion. Averaging lets three
confident answers carry one that nothing addressed — which is precisely the failure being
hunted.

**A low score parks the row; it does not fail a build.** The output is "these criteria look
unaddressed" attached to the row, for a human to agree with or override. A gate that blocks
on a probability teaches people to route around it, and a gate people route around is worse
than none.

**It runs behind the first gate, never instead of it.** Paperwork first, judgement second.
If there is no evidence, there is nothing to assess.

## What it cannot do

Worth stating plainly, because this is the part that gets oversold:

1. **Typed is not true.** Constraining the shape of an answer does not prove the judgement
   inside it. A calibrated 0.93 is a good bet, not a proof.
2. **Thresholds do not transfer.** What counts as passing is empirical per project and per
   model. Any default is a starting point, not a calibration — and
   [reports of where yours settled]({{ site.baseurl }}/CONTRIBUTING.html) are the most
   useful thing anyone can send.
3. **It does not choose the criteria.** If the row says the wrong thing, a confident verdict
   that the build matches the row is a confident yes to the wrong question. This gate
   checks fidelity to intent; it has no opinion about the intent.

So it is a second opinion that never sleeps and has no stake in the work being finished.
That is a modest claim, and it is the honest one.

## Prior art

The approach here is a **System One** decision model — fast, cheap, probabilistic judgement
called inline rather than a deliberative agent, so it can run on every row instead of being
saved for the ones someone already suspects. The underlying idea is
[TypeSafe's Jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev).

The integration pattern is taken from **[foreman](https://github.com/thruwire/foreman)**
(MIT), which connects to the same decision model and is the clearest worked example of
doing it well. Three things worth borrowing from it:

- **An adapter seam rather than a direct dependency.** The process asks for a judgement; a
  swappable adapter decides who answers. A stub adapter makes the whole path testable with
  no network, no credentials and nothing installed — which is what keeps a tool like this
  runnable by someone who has configured none of it.
- **Retry on the transport, not in the logic.** Rate limits and gateway errors are retried
  with backoff that respects what the server asks for. The code deciding whether a row is
  done should never contain the words "try again".
- **Credentials are the client library's business.** The adapter does not read, validate,
  store or log a key. It is handed a configured client or it is switched off. Half-configured
  is an error at startup, not a surprise at the gate.

That is the part of foreman that matters here: how you connect to a decision model without
letting it become load-bearing. Credit where it is due — and
[sending back what you learned]({{ site.baseurl }}/CONTRIBUTING.html) is the behaviour that
makes prior art like that exist in the first place.
