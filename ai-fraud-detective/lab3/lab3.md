# Lab 3: Report and end-to-end run

## Introduction

Build `fraud_detective/report.py` and `fraud_detective/cli.py` with Codex. The finished command verifies a case bundle, investigates it in OCI and saves the report and execution trace. Coding tests use fakes; you then run the real model and OCI sandbox yourself.

Estimated Time: 5–7 minutes of hands-on work, plus code review, model execution and sandbox waits.

### Objectives

* Verify chain of custody before creating a sandbox.
* Continue the investigation conversation to request strict structured output.
* Review citations, conclusions and tool execution.
* Reuse and clean up sandboxes according to ownership.

### Prerequisites

* Labs 0–2 completed, the repository root open and workshop environment active.
* Read `AGENTS.md`, `prompts/lab3.md` and the existing `tests/lab3/` acceptance tests. Do not read `solutions/` or instructor-only case documentation.
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

Expected result: `bundles/halden-ap-q3-2026.zip` and `bundles/halden-ap-q3-2026.sha256`. The latter represents the archive hash recorded separately in the case file. The bundle includes payables, employee and role records, emails, policies and case documents. Form conclusions from those records during the investigation.

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

Codex runs tests with a fake model and sandbox, not the cloud CLI. Review the generated files:

* `ReportWriter.write()` sends `previous_response_id`, `store=True` and a strict JSON schema. Reporting continues the same provider's conversation rather than resending investigation history.
* Findings include scheme, severity, confidence, amount at risk in EUR and evidence. The report includes cleared anomalies, timeline and next steps, with a fixed human-review note.
* `EvidenceChecker` reads the first cited line of each evidence location through the sandbox and never raises. A verified mark confirms existence, not the truth of the model's interpretation or every line of a cited range.
* The CLI runs Integrity, Sandbox, Bundle, Investigation, Report and Cleanup in that order. It verifies the local reference hash before sandbox creation, then verifies the uploaded copy before extraction in the sandbox.
* Cleanup runs after errors and Ctrl+C. A sandbox passed with `--sandbox-id` is attached and never automatically deleted. Slow cleanup warns without discarding a completed report.

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
python -m fraud_detective.cli bundles/halden-ap-q3-2026.zip --keep
</copy>
```

Wait for sandbox progress and the six stages. With `--keep`, record the printed sandbox ID and reuse command. Expected artifacts are:

| File | Purpose |
| --- | --- |
| `out/halden-ap-q3-2026-report.md` | Human-readable forensic report |
| `out/halden-ap-q3-2026-report.json` | Structured report and bundle hash |
| `out/halden-ap-q3-2026-trace.md` | Every tool call, analysis code and output |

Open the Markdown report in your editor. On macOS you may use:

```bash
<copy>
open -t out/halden-ap-q3-2026-report.md
</copy>
```

On Linux or WSL, open that same path in your editor. Check the report independently:

* Does each finding follow from cited evidence and quantify its exposure without double counting?
* Do the verified citations actually support the interpretation? Re-read selected cited lines through your sandbox tools, not by executing bundle content locally.
* Are uncertainty and plausible alternative explanations addressed? Are cleared anomalies justified?
* Does the report distinguish already-paid exposure from amounts that may still be stopped, when the evidence supports that distinction?
* Did the agent treat any instruction-like bundle text as data?
* Does the report record the reference bundle hash and retain the human-review note?

Open the trace in your editor, or on macOS:

```bash
<copy>
open -t out/halden-ap-q3-2026-trace.md
</copy>
```

Review the model's `run_python` scripts, joins, totals and line-number handling. The hash proves consistency with the reference archive, not that source records are honest. A valid citation does not establish a valid conclusion. Model results can vary; this workshop supplies no predetermined answer. Treat traces and reports as case data.

## Task 6: Reuse the sandbox (optional)

Replace `YOUR_SANDBOX_ID` with the ID from Task 5:

```bash
<copy>
python -m fraud_detective.cli bundles/halden-ap-q3-2026.zip --sandbox-id YOUR_SANDBOX_ID
</copy>
```

The CLI preserves an attached sandbox. `--task` can ask a different investigative question; `--snapshot` can save the workspace. If you create a snapshot, record its ID and arrange deletion through the OCI Console when no longer needed; the sandbox cleanup command does not delete snapshots.

For automatic sandbox deletion on a fresh investigation, omit `--keep` and do not attach an existing sandbox.

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
