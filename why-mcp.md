---
title: Why MCP
description: What an MCP server buys over an agent writing its own script — and when it is not worth it.
---

# Why MCP

An agent with shell access does not *need* MCP to edit a CSV. It can `cat tasks.csv`, write
a throwaway script and rewrite the file. **MCP is not what makes this possible.**

Here is what it actually buys, and the honest answer is only one of these three.

<pre class="mermaid">
flowchart TB
    subgraph invented["script invented on the spot"]
      I1["rule re-derived each session"] --> I2["lock written by<br/>the party being gated"] --> I3["arbitrary shell"]
    end
    subgraph tool["MCP tool"]
      M1["rule declared in tools/list,<br/>read before acting"] --> M2["one gate in code:<br/>human CLI and agent<br/>hit the same refusal"] --> M3["nine tools,<br/>matrix only"]
    end
</pre>

## 1 · The rules are written on the box

A jar of cookies with a rule: *you may take one only if you show a note from a grown-up.*
You can tell a kid the rule — but tomorrow there is a different kid.

`tools/list` hands every new session the tool names, schemas and descriptions, including the
sentence saying `done` is refused without a passing validation doc. A fresh agent with no
memory of yesterday reads the rule **before** it tries anything. An agent inventing its own
CSV handling re-derives the column layout every time, and re-derivation is where a column
quietly disappears.

## 2 · The lock is on the jar, not in the kid's head

The gate is Python, in one function. Whether a human types `task-master set US-36 done` or
an agent calls `task_set_status`, the same code refuses. An agent writing its own script to
edit the CSV would be writing its own lock — or, far more likely, skipping it.

That is the exact failure this exists for: **an implementer that can tick off its own work
ticks it off**, and ad-hoc tooling is authored by precisely the party the gate constrains.

## 3 · The door is a small door

Nine tools that only touch the matrix is a much smaller thing to approve than arbitrary
shell. That matters for permission prompts and audit, not for capability.

## The tools

| Tool | Does |
|------|------|
| `task_list` | The whole matrix, with a done count |
| `task_next` | In-progress first, then ready, highest priority |
| `task_show` | One row in full, criteria verbatim |
| `task_set_status` | Move a row — `done` goes through the gate |
| `task_validation` | What evidence exists for a row, and its verdict |
| `task_add` | A row for work that turned up |
| `task_import` | Rows from a pasted markdown table |
| `task_table` | The matrix as markdown, for pasting into a report |
| `task_init` | Create the matrix |

It speaks JSON-RPC on stdio using the **standard library only** — there is nothing to
`pip install`.

```bash
python3 scaffold.py . --mcp     # writes .mcp.json
```

## When MCP is not worth it

A one-off transformation, something you will do once and never again, or a job the ordinary
file tools already cover. The honest reading: **MCP pays for durability of the rule**, not
for the ability to edit a CSV. Take the validation gate away — make the matrix just a list
to read — and the server is mostly ceremony; `cat tasks.csv` would do.

One mechanical note, since "the agent could just invent it" sounds like the two are
interchangeable: a model can write itself a script this turn, but it cannot register an MCP
server into its own running session, because `.mcp.json` is read at startup. Invention gets
you a script now; MCP gets you a tool every turn from now on.

## A process gate, not a necessity

**MCP is a turnstile, not a bridge.** The bridge was always there. A turnstile is worth
installing when every crossing should go through the same check, and is pure overhead when a
person is standing at the gate anyway.

**Context is still a human job.** The matrix holds status, not judgement. Which row matters
this week, whether a criterion is the *right* criterion, whether the thing being built is
what anyone wanted — none of that is in a CSV and no tool call produces it.

**Which is why the documents are thin.** A model reads all of this as language tokens, and
judgement degrades as the pile grows: the more there is to parse at once, the more likely a
detail gets smoothed over, conflated with its neighbour, or confidently invented. So the
chain is many small documents rather than one big one, and each step reads one row, one
contract, one plan.

**And when a human is doing TDD with their own runner, none of this is needed.** They
watched the test go red, then green. They *are* the gate. The MCP tooling is aimed at
workflows with **no human in the loop** — an agent working a backlog overnight, where nobody
is watching it tick the box.

The full-length version of this argument lives in
[ELI5-MCP.md](https://github.com/josepalala-dev/ai-sdlc-tool/blob/master/ELI5-MCP.md).
