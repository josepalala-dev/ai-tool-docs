---
title: Documents, not vibes
description: A cheap, stack-agnostic software lifecycle for working with coding agents — and a done status that needs evidence.
---

# ai-sdlc-tool

{: .lede }
A cheap, stack-agnostic software development lifecycle for working with coding agents.
Every stage is a document on disk, and **nothing is marked done without evidence.**

<div class="cta">
  <a class="primary" href="https://github.com/josepalala-dev/ai-sdlc-tool">View on GitHub</a>
  <a href="{{ site.baseurl }}/the-loop.html">How the loop works</a>
  <a href="{{ site.baseurl }}/task-matrix.html">The task matrix</a>
</div>

## The problem it solves

An agent writes code quickly and reports success confidently. Those two facts together are
the problem: "done" ends up meaning *somebody said so*, usually the same party that did the
work. Scroll back through a chat log three weeks later and you cannot tell which acceptance
criterion was ever actually checked.

This tool makes the lifecycle a set of files instead. The filesystem is the state machine —
no service, no database, no orchestration layer.

## How you use it

<pre class="mermaid">
flowchart TD
    A["task-master next — pick a row"] --> B["/eval — read reality, quote the criteria"]
    B --> C["testing plan — the contract for done"]
    C -->|human agrees| D["row becomes ready"]
    D --> E["/write-plan — numbered steps, no code yet"]
    E -->|human approves| F["/do-it — implement, row in-progress"]
    F --> G["/verify-it — run the gates, grade the contract"]
    G --> H["docs/validation/ — method, verdict, evidence"]
    H --> I{"verdict PASS?"}
    I -->|yes| J["row becomes done"]
    I -->|no| K["row becomes blocked, notes say why"]
    K --> B
    J --> A

    classDef doc fill:#e8f0ec,stroke:#2f6f4f,color:#1a1a1a
    classDef gate fill:#f6e9e0,stroke:#b4621f,color:#1a1a1a
    class C,H doc
    class I,J gate
</pre>

The one rule that is not negotiable: **the contract for "done" is written before the
implementation exists.** Not the tests — the contract. Writing it afterwards means writing
down whatever got built, which grades nothing.

## Start in two minutes

```bash
git clone https://github.com/josepalala-dev/ai-sdlc-tool
python3 ai-sdlc-tool/scaffold.py ~/code/my-project --skills --mcp

cd ~/code/my-project
task-master import --from backlog.md   # a pasted user-story table
task-master next                       # what to work on
```

Python 3 standard library only. No `pip install`, no npm, nothing to keep up to date.

## What it creates

| Path | What lives there |
|------|------------------|
| `AGENTS.md` | The human's description of the project: stack, commands, conventions |
| `docs/dev-tasks/tasks.csv` | **The task matrix** — one row per unit of work, and its status |
| `docs/dev-tasks/` | One file per ticket: user story, acceptance criteria, dependencies |
| `docs/testing-plan/` | The contract for "done", written before the code |
| `docs/arch-docs/` | The design the implementation has to fit |
| `docs/validation/` | The evidence a row is done: method, verdict, criteria checked off |
| `.plans/` | Implementation blueprints. Working scratch, not committed |

## The part that is actually unusual

Most tooling tracks *status*. This tracks *evidence*.

A row reaches `done` only when a file in `docs/validation/` names the task and records a
PASS — with a method that is whatever your project already uses: a unit test, an integration
test, a newman run, a code review, or a written-out manual procedure.

```console
$ task-master set JEV-09 done
error: JEV-09 cannot be done: no validation doc in docs/validation/ names it.
  Write one (see docs/validation/README.md) with a **Task:** JEV-09 header,
  a **Method:** line and a **Verdict:**, then set the status again.
```

An implementer that can tick off its own work ticks it off. So the tooling refuses rather
than trusting — and because the human CLI and the agent's MCP tools run the same code, both
hit the same refusal.

[Read how the gate works →]({{ site.baseurl }}/task-matrix.html)

## Works for agents, not just people

The same operations are exposed over [MCP]({{ site.baseurl }}/why-mcp.html), so an agent
working a backlog unattended reads rows and ticks them off through tools rather than
rewriting a CSV by hand. The rules arrive with the tool descriptions, so a fresh session
with no memory of yesterday reads the gate before it tries anything.

```bash
python3 scaffold.py . --mcp     # registers the server in .mcp.json
```

---

Honest about its limits: the semantic gate that measures *what was built* against *what the
row asked for* is a [prototype]({{ site.baseurl }}/reference.html), the method is not
magic, and choosing which rows matter is still a human job. The rest is text files and a
refusal, which is the cheapest useful thing to standardise across projects an agent touches.
