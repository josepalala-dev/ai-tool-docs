# ai-sdlc-tool — documentation

The published documentation for **ai-sdlc-tool**: a cheap, stack-agnostic software
development lifecycle for working with coding agents, where every stage is a document on
disk and nothing is marked done without evidence.

**Site: [josepalala-dev.github.io/ai-sdlc-docs](https://josepalala-dev.github.io/ai-sdlc-docs)**

## Why this is a separate repository

This repo is the `docs/` submodule of `ai-sdlc-tool`, which is private. GitHub Pages cannot
publish from a private repository on a free plan, and publishing the submodule rather than
the parent is the way round that: the documentation and the task matrix are public, the code
is not.

It also means the site has its own history. A documentation change is one commit here, with
no build step anywhere — GitHub Pages runs Jekyll over the markdown that is already in the
tree.

## What is in here

| Path | What |
|------|------|
| `index.md` | The landing page |
| `the-loop.md` | The five steps, and what each reads and writes |
| `task-matrix.md` | The matrix, its statuses, and the gate on `done` |
| `why-mcp.md` | What MCP buys over an agent writing its own script — and when it doesn't |
| `reference.md` | Links to the canonical documents, and the semantic-gate prototype |
| `arch-docs/` | Design decisions and the reasons behind them |
| `dev-tasks/` | The ticket format, the workflow, and the live task matrix |
| `testing-plan/` | The contract format, and the Jev suite's own contract |
| `validation/` | The evidence format that lets a task be marked done |
| `_layouts/`, `_config.yml` | One layout, inline CSS, mermaid from a pinned CDN |

`dev-tasks/tasks.csv` and `dev-tasks/jev-backlog.md` are excluded from the built site but
kept in the repo: they are the project's working state, and the point of the method is that
the evidence is inspectable.

## Editing

Edit the markdown and push. There is nothing to build or install.

As a submodule, from the parent repository:

```bash
cd docs
git add . && git commit -m "docs: …" && git push
cd ..
git add docs && git commit -m "chore: bump docs submodule"
```

## Contributing and issues

Both live in the tool's repository:
[josepalala-dev/ai-sdlc-tool](https://github.com/josepalala-dev/ai-sdlc-tool) —
[issues](https://github.com/josepalala-dev/ai-sdlc-tool/issues) ·
[contributing](https://github.com/josepalala-dev/ai-sdlc-tool/blob/master/CONTRIBUTING.md).

MIT licensed.
