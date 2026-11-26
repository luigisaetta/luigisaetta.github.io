# Lab 4: Decision support with the Decisions API (optional)

## Introduction

Decision support asks focused questions about evidence already collected. In this optional lab, you will use Codex to build a Python module that sends the saved report to the OpenAI Decisions API and reads probabilities and a suggested action. The OpenAI Responses API performed the investigation in earlier labs; the Decisions API helps investigators assess its results using thresholds they choose themselves, without starting an OCI Sandbox.

Estimated Time: 5–7 minutes of hands-on work, plus reading and review.

### Objectives

* Ask predicates and a bounded choice in a single request.
* Match answers by name and handle refusals.
* Interpret probabilities using thresholds chosen by people.

### Prerequisites

* A completed Lab 3 JSON report at `out/halden-ap-q3-2026-report.json`.
* Workshop environment active, commands run from the repository root.
* An OpenAI API key with access to the source's Decisions API model for the usual path, including when the investigation used OCI.
* Read `AGENTS.md`, `prompts/lab4.md` and `tests/lab4/test_decisions.py`.

## Task 1: Configure the decision provider

The decision provider is independent of `[model]`. Edit the existing `[decisions]` section in your participant checkout's `config/settings.toml`:

```toml
[decisions]
provider = "openai"
model_id = "gpt-6-luna"
```

Keep `OPENAI_API_KEY` in your private `.env`. For the OCI alternative, set `provider = "oci"`, `model_id = "openai.gpt-6-luna"`, configure `[oci] region` and set `OCI_GENAI_API_KEY`. Confirm the required OCI API-key permissions with your administrator.

Both paths rejoin at Task 2 and use the same test and run commands. Confirm that your provider offers the Decisions API and selected model; if unavailable, you can finish the workshop after Lab 3.

## Task 2: See the tests fail

```bash
<copy>
python -m pytest tests/lab4 -q
</copy>
```

On a fresh checkout expect `ModuleNotFoundError: fraud_detective.decisions`.

## Task 3: Ask Codex to build decision support

```text
<copy>
Read prompts/lab4.md and do what it says (everything below the first line). Follow AGENTS.md; do not read solutions/ or instructor-only case documentation. Do not call the real API during implementation.
</copy>
```

Codex builds `fraud_detective/decisions.py` using a fake client for tests; this module reads only the saved report and creates no sandbox. Review the implementation:

* It uses the provided `DecisionsClientFactory`, which handles the separate provider settings and omits the project header on OCI.
* It reads only the saved JSON report and sends only `REPORT_FIELDS` as `input_json`.
* One request asks two predicates per finding: whether evidence indicates a deliberate act, and whether any at-risk amount is unpaid and can still be stopped. A final choice asks for the next action for the case.
* Answers are matched by `name`, not position. Refusals become `None`; missing answers produce a clear error.
* `LIKELY = 0.7` and `UNLIKELY = 0.3` are explicit human-selected thresholds, and output always includes the fixed decision note.

The API returns indicators rather than explanatory reasoning. The saved report supplies the evidential context; investigators retain responsibility for action. Probabilities depend on the quality of that report and the wording of each question.

## Task 4: Run the checks

```bash
<copy>
python -m pytest tests/unit tests/lab4 -q
python -m black --check fraud_detective tests
python -m pylint fraud_detective
</copy>
```

Expected result: all tests pass and formatting/lint checks report no problems.

## Task 5: Run it on your report

This sends report fields to your separately configured decision provider:

```bash
<copy>
python -m fraud_detective.decisions out/halden-ap-q3-2026-report.json
</copy>
```

Expected result: one block per finding and a suggested next action; `out/halden-ap-q3-2026-decisions.md` and `out/halden-ap-q3-2026-decisions.json` are written.

Check every indicator against your report. Does the report contain enough evidence for each question? Is the proposed action consistent with the reported facts and uncertainty? Treat refusals and intermediate probabilities explicitly.

## Task 6: Explore question wording (optional)

In `build_questions`, try combining "stopped or recovered" in place of the unpaid-amount question. Run Task 5 again and compare the answers with the precise claims in your own report. The revised wording mixes a payment not yet made with recovery after payment. Restore the original question and rerun the tests.

Expected result: you can explain the difference between open-ended investigation and bounded decision support, why questions need observable criteria, and who selects thresholds. No sandbox cleanup is needed for this module; ensure Lab 3 cleanup is complete and retain decision files only as needed. Use [Need Help](../workshops/index.html?lab=need-help) for recovery.

## Acknowledgements

* **Author** - L. Saetta
* **Last Updated By/Date** - L. Saetta, November 2026
