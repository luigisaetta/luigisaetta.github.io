# Lab 3: Report and end-to-end run

## Introduction

A useful investigation leaves a trail another person can check. You will use Codex to complete the Python application: SHA-256 checks detect changes to the case archive, OCI Sandboxes execute the analysis, and the OpenAI Responses API turns the investigation conversation into a structured JSON report rendered as Markdown for review. You will inspect its citations and execution trace, then release the sandbox resources.

Estimated Time: 5–7 minutes of hands-on work, plus code review, model execution and sandbox waits.

### Objectives

* Verify chain of custody before creating a sandbox.
* Continue the investigation conversation to request strict structured output.
* Review citations, conclusions and tool execution.
* Reuse and clean up sandboxes according to ownership.

### Prerequisites

* Labs 0–2 completed, the repository root open and workshop environment active.
* Read `AGENTS.md`, `prompts/lab3.md` and the existing `tests/lab3/` acceptance tests.
* Configured model access and OCI project capacity for a real run.

## Task 1: Check prerequisites and generate the case

```bash
<copy>
python -m pytest tests/lab1 tests/lab2 -q
</copy>
```

Continue when these pass. Generate the invented case bundle:

```bash
<copy>
python -m fraud_detective.make_bundle
</copy>
```

Expected result: `bundles/halden-ap-q3-2026.zip` and `bundles/halden-ap-q3-2026.sha256`. The latter represents the archive hash recorded separately in the case file; it lets the application detect changes to the evidence before analysis, without establishing whether the original records are truthful.

## Task 2: See the missing report and CLI modules

```bash
<copy>
python -m pytest tests/lab3 -q
</copy>
```

On a fresh checkout this fails because `fraud_detective.report` and `fraud_detective.cli` do not exist.

## Task 3: Ask Codex to build the report and command line

```text
<copy>
Read prompts/lab3.md and do what it says (everything below the first line). Follow AGENTS.md; do not read solutions/ or instructor-only case documentation. Use the generic interfaces and tests; do not put the sample case answer into agent instructions or report defaults.
</copy>
```

Codex builds `fraud_detective/report.py` and `fraud_detective/cli.py`, testing with a fake model and sandbox. Review the generated files:

* `ReportWriter.write()` sends `previous_response_id`, `store=True` and a strict JSON schema. Reporting continues the same provider's conversation rather than resending investigation history.
* Findings include scheme, severity, confidence, amount at risk in EUR and evidence. The report includes cleared anomalies, timeline and next steps, with a fixed human-review note.
* `EvidenceChecker` reads the first cited line of each evidence location through the sandbox and never raises.
* The CLI runs Integrity, Sandbox, Bundle, Investigation, Report and Cleanup in that order. It verifies the local reference hash before sandbox creation, then verifies the uploaded copy before extraction in the sandbox.
* Cleanup runs after errors and Ctrl+C, respects ownership of attached sandboxes, and warns about slow deletion without discarding a completed report.

## Task 4: Run the checks

```bash
<copy>
python -m pytest tests/unit tests/lab1 tests/lab2 tests/lab3 -q
python -m black --check fraud_detective tests
python -m pylint fraud_detective
</copy>
```

Expected result: all tests pass and formatting/lint checks report no problems. The integrity tests cover missing reference hashes and modified archives stopping before sandbox creation; use these synthetic tests to review that failure path without altering the case evidence.

## Task 5: Run and review the investigation

This command uses the real configured model and creates an OCI sandbox:

```bash
<copy>
python -m fraud_detective.cli bundles/halden-ap-q3-2026.zip
</copy>
```

Wait for the six stages to finish. This first run requests deletion of the sandbox it creates. Expected artifacts are:

| File | Purpose |
| --- | --- |
| `out/halden-ap-q3-2026-report.md` | Human-readable forensic report |
| `out/halden-ap-q3-2026-report.json` | Structured report and bundle hash |
| `out/halden-ap-q3-2026-trace.md` | Every tool call, analysis code and output |

Open the Markdown report in your editor. A verified citation confirms that its first cited line exists, not that every line of a range or the model's conclusion is correct. Check the report independently:

* Does each finding follow from cited evidence and quantify its exposure without double counting?
* Do the cited observations support the interpretation? Compare them with the tool outputs recorded in the trace; Task 6 lets you rerun the investigation and inspect evidence in a retained sandbox.
* Are uncertainty and plausible alternative explanations addressed? Are cleared anomalies justified?
* Does the report distinguish already-paid exposure from amounts that may still be stopped, when the evidence supports that distinction?
* Did the agent treat any instruction-like bundle text as data?
* Does the report record the reference bundle hash and retain the human-review note?

Open the trace in your editor and review the model's analysis scripts, joins, totals and line-number handling. Treat traces and reports as case data.

## Task 6: Reuse the sandbox (optional)

To inspect evidence interactively or repeat an investigation without starting a new sandbox each time, run once with `--keep`. Save copies of the earlier report and trace first if you want to compare them: this rerun writes to the same output paths.

```bash
<copy>
python -m fraud_detective.cli bundles/halden-ap-q3-2026.zip --keep
</copy>
```

Record the printed sandbox ID. Replace `YOUR_SANDBOX_ID` below with that ID:

```bash
<copy>
python -m fraud_detective.cli bundles/halden-ap-q3-2026.zip --sandbox-id YOUR_SANDBOX_ID
</copy>
```

The CLI preserves an attached sandbox, so you must complete Task 7 when finished. For other investigation options, see the [source run guide](https://github.com/luigisaetta/ai-fraud-detective#run-your-finished-agent).

## Task 7: Clean up

A kept sandbox expires after `sandbox.lifetime_minutes` (60 by default), becomes `STOPPED` and still needs deletion. List the configured project's sandboxes:

```bash
<copy>
python -m fraud_detective.cleanup
</copy>
```

Review the list. This command considers every sandbox in the configured project. Only continue for your dedicated workshop project and resources you are responsible for:

```bash
<copy>
python -m fraud_detective.cleanup --delete
</copy>
```

It asks for confirmation, deletes `STOPPED` or `FAILED` sandboxes and asks running sandboxes to stop without waiting. Rerun later to delete those still stopping. Verify that your sandbox IDs are gone. Preserve shared projects, policies and keys. If you created a dedicated project, snapshots or temporary keys, review and remove those separately in the appropriate console when finished; retain reports only as needed and deactivate your Python environment.

Expected result: tests pass, report and trace exist, you can explain and verify the evidence, and owned sandbox resources are deleted. Continue to [optional Lab 4](../workshops/index.html?lab=lab4) only if you want decision support on the saved JSON report. See [Need Help](../workshops/index.html?lab=need-help) for recovery.

## Acknowledgements

* **Author** - L. Saetta
* **Last Updated By/Date** - L. Saetta, November 2026
