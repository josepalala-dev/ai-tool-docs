---
title: Why MCP
description: What an MCP server buys over an agent writing its own script — and when it is not worth it.
---

# Why MCP

An agent with shell access does not *need* MCP to edit the matrix. It can read the file, write
a throwaway script and rewrite the file. **MCP is not what makes this possible.**

Here is what it actually buys, and the honest answer is only one of these three.

<pre class="mermaid">
flowchart TB
    subgraph invented["a script invented on the spot"]
      direction TB
      I1["the rule is re-derived each session"] --> I2["the lock is written by the party being gated"]
      I2 --> I3["arbitrary shell to approve"]
    end
    subgraph tool["an MCP tool"]
      direction TB
      M1["the rule is declared in tools/list, read before acting"] --> M2["one gate in code: human CLI and agent hit the same refusal"]
      M2 --> M3["nine tools, matrix only"]
    end
</pre>

## 1 · The rules are written on the box

A jar of cookies with a rule: *you may take one only if you show a note from a grown-up.*
You can tell a kid the rule — but tomorrow there is a different kid.

`tools/list` hands every new session the tool names, schemas and descriptions, including the
sentence saying `done` is refused without a passing validation doc. A fresh agent with no
memory of yesterday reads the rule **before** it tries anything. An agent inventing its own
Hand-editing re-derives the layout every time, and re-derivation is where a field
quietly disappears.

## 2 · The lock is on the jar, not in the kid's head

The gate is one rule in one place. Whether a human moves a row or
an agent calls `task_set_status`, the same code refuses. An agent writing its own script to
edit the record directly would be writing its own lock — or, far more likely, skipping it.

That is the exact failure this exists for: **an implementer that can tick off its own work
ticks it off**, and ad-hoc tooling is authored by precisely the party the gate constrains.

## 3 · The door is a small door

Nine tools that only touch the matrix is a much smaller thing to approve than arbitrary
shell. That matters for permission prompts and audit, not for capability.

## The tools

| Tool | Does |
|------|------|
| Read the matrix | The whole thing, with a done count |
| Ask what is next | In-progress first, then ready, highest priority |
| Read one row | Criteria verbatim, never truncated |
| Move a row | And `done` goes through the gate |
| Ask what evidence exists | For a given row, with its verdict |
| Add a row | For work that turned up |
| Import rows | From a pasted user-story table |

Each one is deliberately narrow. The agent is not handed a file to rewrite; it is handed
the few questions and moves the process actually has.

## When MCP is not worth it

A one-off transformation, something you will do once and never again, or a job the ordinary
file tools already cover. The honest reading: **MCP pays for durability of the rule**, not
for the ability to edit a file. Take the validation gate away — make the matrix just a list
to read — and the protocol is mostly ceremony; reading the file would do.

One mechanical note, since "the agent could just invent it" sounds like the two are
interchangeable: a model can write itself a script this turn, but it cannot register an MCP
server into its own running session, because that configuration is read at startup. Invention gets
you a script now; MCP gets you a tool every turn from now on.

## A process gate, not a necessity

**MCP is a turnstile, not a bridge.** The bridge was always there. A turnstile is worth
installing when every crossing should go through the same check, and is pure overhead when a
person is standing at the gate anyway.

**Context is still a human job.** The matrix holds status, not judgement. Which row matters
this week, whether a criterion is the *right* criterion, whether the thing being built is
what anyone wanted — none of that is in the matrix and no tool call produces it.

**Which is why the documents are thin.** A model reads all of this as language tokens, and
judgement degrades as the pile grows: the more there is to parse at once, the more likely a
detail gets smoothed over, conflated with its neighbour, or confidently invented. So the
chain is many small documents rather than one big one, and each step reads one row, one
contract, one plan.

**And when a human is doing TDD with their own runner, none of this is needed.** They
watched the test go red, then green. They *are* the gate. The MCP tooling is aimed at
workflows with **no human in the loop** — an agent working a backlog overnight, where nobody
is watching it tick the box.

What a gate like that cannot check — whether the evidence is about the right row — is
[the second gate]({{ site.baseurl }}/second-gate.html), which is a design rather than a
feature.
