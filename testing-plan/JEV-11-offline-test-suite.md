# Testing plan — JEV-11 offline test suite for the Jev semantic gate

**Written:** 2026-10-31 (before implementation)
**Ticket:** `docs/dev-tasks/jev-backlog.md` — rows JEV-01 … JEV-11

The durable form of `.plans/2026-10-31-jev-unit-tests.md`, which is scratch. This is the
contract `/verify-it` grades against, and the rows below are the ones a validation doc has
to check off one by one.

## Scope

What this change can break:

- `decision_model.py` — the provider seam and the Jev adapter (new).
- `assess.py` — row to questions, state assembly, the threshold decision (new).
- `task_master.set_status` — the single chokepoint for all five statuses. A bug here blocks
  every status change, not only `done`.
- `mcp_server.call_tool` — the sync dispatch an async SDK has to be called from.
- Everyone with no provider configured: the stdlib-only promise and today's behaviour must
  both survive untouched.

## Behaviours

| # | Behaviour that must hold | How it is checked | Type |
|---|--------------------------|-------------------|------|
| 1 | The adapter returns one probability per question id | `tests/test_decision_model.py::JevAdapterHappyPath::test_returns_a_probability_per_question` | unit |
| 2 | Every question for a row goes in a single call | `…::test_sends_every_question_in_one_call` | unit |
| 3 | Every call carries a bounded deadline | `…::test_passes_the_state_and_a_bounded_timeout` | unit |
| 4 | The result says which provider and model graded it | `…::test_identifies_its_provider_and_model` | unit |
| 5 | Finite values outside `[0, 1]` are clamped, not rejected | `…::ValueParsing::test_clamps_into_the_unit_interval` | unit |
| 6 | Both response shapes parse: `.nouls[id].noul` and a plain mapping | `…::ValueParsing` (both tests) | unit |
| 7 | With nothing configured, the gate is off and no call is attempted | `…::MisconfiguredToken::test_nothing_configured_means_the_gate_is_off` | unit |
| 8 | One acceptance-criteria cell becomes one question per clause | `tests/test_assess.py::QuestionsPerCriterion` | unit |
| 9 | Each question carries its criterion verbatim and reads so high means yes | `…::test_each_question_carries_its_criterion_verbatim`, `…::test_a_question_is_phrased_so_high_means_yes` | unit |
| 10 | State carries the criteria, the diff and the named artifacts' contents | `tests/test_assess.py::StateAssembly` | unit |
| 11 | A pass requires every criterion over the threshold | `…::TheDecision::test_all_criteria_above_threshold_passes` | unit |
| 12 | A failing criterion is named, not averaged away | `…::test_one_low_criterion_fails_and_is_named` | unit |
| 13 | The result records provider, threshold and date for audit | `…::test_the_result_records_what_graded_it`, `…::test_the_summary_line_is_auditable_on_the_row` | unit |
| 14 | `done` needs the paperwork *and* the assessment | `tests/test_gate.py::BothKeysNeeded` | unit |
| 15 | The refusal names which of the two keys failed | `…::test_the_refusal_says_which_of_the_two_keys_failed` | unit |
| 16 | A high score never substitutes for a missing validation doc | `…::PaperworkStillRequired` | unit |
| 17 | A low assessment can park the row as `blocked` with the criterion in its notes | `…::ParksRatherThanFails` | unit |
| 18 | With no assessor, behaviour is exactly today's and the column stays empty | `…::OffUnlessConfigured` (first two tests) | unit |
| 19 | `force` still bypasses both keys and records that it did | `…::test_force_still_bypasses_both_keys` | unit |
| 20 | The four non-`done` statuses never consult the model | `…::OtherStatusesUntouched` | unit |

Run the suite with:

```bash
python3 -m unittest discover -s tests -t . -v
```

## Fixtures and setup

- **No credentials.** Nothing in here may read `TYPESAFE_API_KEY` from the real environment;
  `adapter_from_env` is handed an explicit dict.
- **No network.** `tests/fakes.py` supplies `FakeJevClient`, `TimingOutClient` and
  `MalformedClient`. A test that would call out is a failed test.
- **No SDK.** `typesafe-sdk` is not installed here, so the absent-SDK path is the default
  one and is exercised for free.
- Matrix and validation fixtures are written into a `TemporaryDirectory` per test, so no
  test touches this repo's own `tasks.csv`.
- `python3 -m unittest` from the repo root; `tests/__init__.py` makes the package resolve.

## Negative cases

| # | Input or state | Expected handling |
|---|----------------|-------------------|
| 1 | No provider and no token | `adapter_from_env` returns `None`; the gate is off; nothing is logged as an error |
| 2 | Provider set, `TYPESAFE_API_KEY` absent | `DecisionModelError` naming the variable |
| 3 | Token present but blank or whitespace | `DecisionModelError` — treated as missing, not as a key |
| 4 | Token still a placeholder (`your-api-key-here`, `sk-xxxxxxxx`, `changeme`, `<token>`) | `DecisionModelError` whose message contains "placeholder" |
| 5 | Unknown provider name (e.g. `jevv`) | `DecisionModelError` naming the providers that do exist |
| 6 | Token set but no provider | `None` — a stray key must not switch the gate on |
| 7 | `typesafe-sdk` not installed | `DecisionModelError` mentioning typesafe, raised at call time, not import time |
| 8 | Provider call times out | `DecisionModelError`; row unchanged |
| 9 | Response missing an answer | `DecisionModelError` naming the missing question id |
| 10 | Response non-numeric, empty, `None`, or a bare string | `DecisionModelError`; **never** a defaulted score |
| 11 | Acceptance criteria empty or whitespace | `AssessmentError` — an unassessable row is refused, not passed |
| 12 | Named artifact missing from disk | Listed in `state["missing_artifacts"]`; the call still proceeds |
| 13 | Named artifact outside the project root | `AssessmentError` — a row cannot pull in arbitrary files |
| 14 | Provider error during a `done` transition | Error propagates; row keeps its previous status |

## Not covered

- **Anything against the live Jev service.** No key and no SDK in this environment, and
  `docs.typesafe.ai` is unreachable from here. The adapter's exact call signature is taken
  from foreman's `src/foreman/decision_models/jev.py` and is **unverified** — the fakes
  encode that assumption in one place so it is cheap to correct.
- **Calibration.** Whether `0.7` is the right threshold, or whether a 0.93 means what it
  says, is a per-project empirical question. These tests prove the plumbing, not the
  judgement, and no row should be closed on the strength of them alone.
- **Async transport.** The real SDK is `AsyncTypeSafeClient`; the fakes are synchronous.
  The `asyncio.run` boundary in the MCP server's sync dispatch is untested and is its own
  row's problem.
