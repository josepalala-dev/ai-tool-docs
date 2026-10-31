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

## Contributing

Contribution happens here, not in the private repository: see
[CONTRIBUTING.md](CONTRIBUTING.md). Documentation takes pull requests normally. Changes to
behaviour are filed as issues — described as a matrix row — because the implementation is
not published.

## What is in here

The site is the handful of narrative pages. Everything else is the parent project's working
state, kept in the repo but **excluded from the published site** — it describes how the
tooling is built, and the site is deliberately about the process instead.

| Path | Published | What |
|------|-----------|------|
| `index.md` | yes | The landing page |
| `the-loop.md` | yes | The five steps, and what each reads and writes |
| `task-matrix.md` | yes | The matrix, its statuses, and the gate on `done` |
| `why-mcp.md` | yes | What MCP buys over an agent writing its own script — and when it doesn't |
| `second-gate.md` | yes | The design for measuring built-vs-asked. A design, not a feature |
| `CONTRIBUTING.md` | yes | What is worth sending, given that the source is private |
| `_layouts/`, `_config.yml` | — | One layout, inline CSS, mermaid from a pinned CDN |
| `arch-docs/` | **no** | Design decisions and the reasons behind them |
| `dev-tasks/` | **no** | The ticket format, the workflow, and the live task matrix |
| `testing-plan/` | **no** | The contract format, and the gate suite's own contract |
| `validation/` | **no** | The evidence format that lets a task be marked done |

Those four folders have to stay here rather than move back: the tooling reads them through
this submodule at `docs/`, so deleting them would break the project that consumes them.
Excluding them from the site is the way to keep both — the tool keeps working, and the
published pages stay about the method rather than its internals.

Keep it that way when adding a page. The site explains *the process*; it does not document
the code, name its modules, or print its commands.

## Editing

Edit the markdown and push. There is nothing to build or install.

As a submodule, from the parent repository:

```bash
cd docs
git add . && git commit -m "docs: …" && git push
cd ..
git add docs && git commit -m "chore: bump docs submodule"
```

MIT licensed.
