# Decision model

**Last reviewed:** 2026-10-31
**Status:** prototype — the transport is unverified against the provider's own docs

## Overview

The semantic gate needs a model that answers a narrow question cheaply: *does this diff
address this acceptance criterion?* That is a judgement, so it cannot be a regex; it runs on
every row, so it cannot be a chat model.

A **System One model** fits: structured state and typed questions in, probabilities out.
The provider is [TypeSafe's Jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev).
Everything in `decision_model.py` sits behind a seam so the tool keeps working with no
provider, no SDK and no key — which is the normal case.

## Flow

```
assess.py                     decision_model.py                    typesafe-sdk
─────────                     ─────────────────                    ────────────
row + diff + artifacts
   │
   ├─ questions_for(row) ──▶ {c1: "…?", c2: "…?"}
   ├─ build_state(...)  ──▶ {criteria, diff, artifacts, missing_artifacts}
   │
   └──────────────────────▶ adapter.evaluate(state, questions)
                                  │
                                  ├─ _wrap:  {c1: Noul(instructions=…)}
                                  ├─ _call:  system_one(state, questions, model, timeout) ──▶ AsyncTypeSafeClient
                                  ├─ _run:   await from sync code (MCP dispatch is sync)
                                  └─ parse_noul_values(response, keys)
                                        response.nouls[c1].noul  ──▶ 0.93
                                               │
                              DecisionResult({c1: 0.93, c2: 0.21}, raw)
                                               │
   Assessment(scores, threshold, unaddressed, provider, model, date)  ◀──┘
```

## Contracts

**Adapter** — anything with this shape is a provider. Implemented by `JevAdapter` and
`StubAdapter`:

```python
provider: str                 # "jev", "stub" — recorded on the row
model: str                    # "jev-latest" — recorded on the row
evaluate(state, questions) -> DecisionResult
close() -> None
```

**`DecisionResult.values[id]`** is **how likely that statement is to be true** — not how
confident the model is about a label it chose. Everything downstream compares it to a
threshold, so confusing the two would make the threshold meaningless.

**Questions** are phrased so a high value means yes. That is what makes Jev's Noul primitive
the direct representation and keeps `min(scores) >= threshold` a legible rule.

**Response shapes accepted** — two, because the SDK returns objects and a proxy may return
plain JSON:

| Shape | Read as |
|-------|---------|
| `response.nouls[id].noul` | the SDK's own objects |
| `{"answers": {id: {"noul": 0.9}}}` or `{id: 0.9}` | proxied or stubbed JSON |

Finite values are clamped into `[0, 1]`. A missing, non-numeric, non-finite or unreadable
answer raises `DecisionModelError`.

**Configuration** — all environment, no config file:

| Variable | Read by | Meaning |
|----------|---------|---------|
| `TASKMASTER_DECISION_PROVIDER` | us | `jev` or `stub`. **Unset means the gate is off** |
| `TASKMASTER_DECISION_MODEL` | us | defaults to `jev-latest` |
| `TASKMASTER_DECISION_TIMEOUT` | us | seconds; defaults to 10 |
| `TASKMASTER_ASSESS_THRESHOLD` | us | the bar every criterion must clear |
| `TYPESAFE_API_KEY` | **the SDK** | we only preflight that it exists and is not a placeholder |
| `TYPESAFE_BASE_URL` | **the SDK** | set it to `https://openrouter.ai/api` to route through OpenRouter |

## Decisions

| Decision | Reason | Date |
|----------|--------|------|
| A System One model, not a chat model | Three properties we need and chat does not have: a declared answer shape, a calibrated number, and a price that survives running on every row every time | 2026-10-31 |
| Noul (a probability), not Choice or Score | Every check is phrased so high means yes, which makes the probability the direct answer and keeps the threshold rule readable | 2026-10-31 |
| An adapter protocol rather than calling the SDK directly | Jev is one provider of this shape. The seam is also what lets the whole gate be tested with no key | 2026-10-31 |
| Credentials are the SDK's business | It already reads `TYPESAFE_API_KEY` / `TYPESAFE_BASE_URL`, and that is exactly what makes the OpenRouter route work with no code change. We preflight only so a missing key reads as a sentence, not a 401 | 2026-10-31 |
| Placeholder keys are rejected by name | `your-api-key-here` left in a `.env` is a real and frequent failure; a readable error beats a provider 401 | 2026-10-31 |
| `adapter_from_env` returns `None` when unconfigured, but raises when half-configured | Off is a supported mode. Silently skipping a check someone believes is running is not | 2026-10-31 |
| A stray `TYPESAFE_API_KEY` does not switch the gate on | A key in the environment for another tool must not start billing this one | 2026-10-31 |
| Fail closed: every provider failure raises, nothing defaults | A fabricated probability would close a row with evidence nobody produced — the precise failure the gate exists to prevent | 2026-10-31 |
| A missing answer for one question fails the whole call | Dropping it silently would pass a criterion nobody graded | 2026-10-31 |
| `StubAdapter` reports `provider="stub"` | A row graded by a stub must never look model-graded on the audit trail | 2026-10-31 |
| The adapter is sync-facing, with `asyncio.run` inside | The SDK is async; the MCP dispatch is sync. One boundary, in one place, instead of an async colouring the whole codebase | 2026-10-31 |
| `_wrap` falls back to plain strings when the SDK is absent | It is what lets an injected fake be tested on a machine with no `typesafe-sdk`. The real path is unchanged | 2026-10-31 |
| `typesafe-sdk` stays an optional extra | The README promises standard library only, in three places. Breaking that for a feature most users will not enable is a bad trade | 2026-10-31 |

## Provenance and its limits

The transport is modelled on [foreman](https://github.com/thruwire/foreman) (MIT),
`src/foreman/decision_models/{base,jev}.py`, which drives the same SDK for a different job:
it supervises a *running* coding agent (is the worker stuck, is it unsafe), where we ask one
question about a *finished* row. No code was copied — what transfers is how you connect,
which the SDK fixes rather than us choosing.

⚠️ **The call signature is unverified.** `docs.typesafe.ai` is unreachable from this
machine, so `system_one(state=…, questions={id: Noul(instructions=…)}, model=…, timeout=…)`
and the `.nouls[id].noul` read are taken on foreman's authority. If they are wrong, they are
wrong in `JevAdapter._call`, `parse_noul_values` and `tests/fakes.py` — deliberately three
places, all of them small.

⚠️ **Typed is not true.** The schema constrains the shape of the answer; it does not prove
the judgement inside it. A calibrated 0.93 is a good bet, not a proof.

⚠️ **Thresholds are not transferable.** What counts as a passing probability is an empirical
question per project and per model. A default is a starting point, not a calibration, and
nothing here establishes that any provider's numbers are accurate.

## Constraints

- `task_master` must not import `decision_model` at module load.
- No module here may read the real environment during tests; `adapter_from_env` takes an
  explicit mapping.
- No network in the test suite. A test that would call out is a failed test.
- Never log or persist the API key, and never put a raw provider response in the matrix.
- Every call carries an explicit deadline. A call with no timeout can hang the gate.
