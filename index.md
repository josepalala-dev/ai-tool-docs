---
title: Documents, not vibes
description: A cheap, stack-agnostic software lifecycle for working with coding agents — and a done status that needs evidence.
---

# ai-sdlc-tool

{: .lede }
A cheap, stack-agnostic software development lifecycle for working with coding agents.
Every stage is a document on disk, and **nothing is marked done without evidence.**

<div class="cta">
  <a class="primary" href="{{ site.baseurl }}/the-loop.html">How the loop works</a>
  <a href="{{ site.baseurl }}/task-matrix.html">The task matrix</a>
  <a href="https://github.com/josepalala-dev/ai-tool-docs">This site on GitHub</a>
</div>

{: .note }
**The method is public; the implementation is not.** Everything that makes this work — the
loop, the matrix, the gate and the document formats — is written down here and MIT
licensed. The code that implements it is a private repository. The formats are the spec, so
reimplementing them in your own stack is a legitimate use of them, and
[contributing]({{ site.baseurl }}/CONTRIBUTING.html) explains
what that means in practice.

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
    A["pick a row from the matrix"] --> B["/eval — read reality, quote the criteria"]
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

## What it leaves behind

Each stage writes one kind of document, and together they are the whole record:

| The artefact | What it settles |
|---|---|
| A project description | The stack, the commands, the conventions — written by the human |
| **The task matrix** | One row per unit of work, its status, and what it traces back to |
| A ticket | The user story and its acceptance criteria |
| A testing plan | The contract for "done", agreed before the code exists |
| A design note | The shape the implementation has to fit |
| **A validation record** | The evidence: method, verdict, every criterion checked off |

Three weeks later the question "was this criterion ever actually checked?" has an answer,
and the answer is a file rather than a memory.

## The part that is actually unusual

Most tooling tracks *status*. This tracks *evidence*.

A row reaches `done` only when a validation record names that task and carries a PASS —
by a method that is whatever your project already uses: a unit test, an integration test,
an API run, a code review, or a written-out manual procedure.

```console
$ set JEV-09 done
error: JEV-09 cannot be done — no validation record names it.
  Write one naming the task, the method and the verdict, then set the status again.
```

An implementer that can tick off its own work ticks it off. So the tooling refuses rather
than trusting — and the human and the agent hit the same refusal, because it is the same
rule rather than two copies of it.

[Read how the gate works →]({{ site.baseurl }}/task-matrix.html)

## Works for agents, not just people

The same operations are exposed over [MCP]({{ site.baseurl }}/why-mcp.html), so an agent
working a backlog unattended moves rows through the same gate rather than editing the
record by hand. The rules arrive with the tools, so a fresh session with no memory of
yesterday meets the gate before it tries anything.

---

Honest about its limits: the second gate — the one that measures *what was built* against
*what the row asked for* — is a prototype, the method is not magic, and choosing which rows
matter is still a human job. The rest is documents and a refusal, which is the cheapest
useful thing to standardise across projects an agent touches.
