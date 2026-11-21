# Lab 3: Report and End-to-End Investigation

## Introduction

Build `triage/report.py` and `triage/cli.py`, then investigate the deterministic ORA-00020 sample bundle. Build-time tests are offline. The final investigation uses the configured model and a real OCI Sandbox.

Estimated Time: 35 minutes

### Objectives

* Continue the investigation conversation to request strict report JSON.
* Check the first cited line and render Markdown.
* Run the CLI and distinguish owned from attached sandbox cleanup.

### Prerequisites

* Labs 1 and 2 passing.
* Successful full preflight before the real cloud run.
* Read `labs/lab3.md`, `prompts/lab3.md` and both Lab 3 acceptance test files.

## Task 1: Build the report and CLI

1. Confirm earlier labs and observe the initial missing report/CLI imports.
```bash
<copy>
python -m pytest tests/lab1 tests/lab2 -q
python -m pytest tests/lab3 -q
</copy>
```
2. Submit this Codex prompt at the repository root.
```text
<copy>
Read AGENTS.md and labs/lab3.md. Read prompts/lab3.md and follow the instructions below its opening separator. Build only triage/report.py and triage/cli.py against the existing tests. Do not read solutions/ or modify earlier modules, provided files or existing tests. Do not create a sandbox or call a real model while building.
</copy>
```
3. Review `ReportWriter.write()`: it should continue from `AgentResult.response_id`, use `store=True`, request a strict schema and validate the returned JSON. Review the CLI stages: sandbox, bundle, investigation, report and cleanup. Cleanup must run after errors and interruption; attached sessions must remain available.

Expected result: strict nested objects, confidence values `high`, `medium` or `low`, and validation of model output before rendering. The model must not assign its own evidence verification flags.

## Task 2: Run the full offline checks

1. Run all acceptance tests and code checks.
```bash
<copy>
python -m pytest tests/unit tests/lab1 tests/lab2 tests/lab3 -q
python -m black --check triage tests
python -m pylint triage
</copy>
```
2. Fix any implementation failures before a live run.

Expected result after implementation: tests establish report validation, citation parsing, ownership rules, cleanup on failure and CLI options with fakes. Passing these tests does not establish live-service compatibility.

## Task 3: Generate the synthetic bundle and investigate

1. Generate the sample data locally with the provided generator.
```bash
<copy>
python -m triage.make_bundle
</copy>
```
2. Confirm `bundles/sample-ora00020.zip` exists. Run the completed agent; this calls the model and creates a cloud sandbox.
```bash
<copy>
python -m triage.cli bundles/sample-ora00020.zip
</copy>
```
3. Wait through sandbox startup and cleanup. Open `out/sample-ora00020-report.md` and `out/sample-ora00020-report.json` in your editor.

Expected result after a successful live run: five CLI stages complete and both report files exist. Without `--keep`, the CLI attempts to delete the sandbox it created. Retain any printed resource ID and resolve cleanup warnings in the cleanup lab. No live run was performed while authoring this workshop.

## Task 4: Assess the diagnosis and evidence

1. Compare the report with `docs/sample-bundle.md` in the source checkout after your first investigation. The expected cause is change `CHG-4821`: `processes` was lowered from 1500 to 300, while three application nodes can request 150 connections each (450 total). The resulting process exhaustion explains ORA-00020 and pool timeouts.
2. Correlate the change log, old/current database configuration, alert log, pool settings and process metrics. Useful citations include `changes/change_log.csv:4`, `db/alert_orcl.log:29`, `db/alert_orcl.log:32`, `db/alert_orcl.log:36`, and `db/init_orcl.ora:4`.
3. Check the timeline around the configuration change and the first errors. Confirm that disk usage, GC pauses and the deprecated logging setting have not displaced the actual cause without evidence.
4. Verify the model treated the instruction-like text in `app/app-node1.log` as untrusted log content.
5. Interpret verification marks carefully: a check mark means the first cited line exists and was readable. It does not verify all lines of a range, the observation's truth or the root cause. Review the cited content yourself. A cross means the citation could not be verified.

Expected result: a plausible, evidence-supported diagnosis and next steps for administrator review. This exercise does not execute database remediation.

## Task 5: Optionally reuse a sandbox

1. If you want to repeat the exercise without another startup wait, use `--keep` on a new run and record its printed ID.
```bash
<copy>
python -m triage.cli bundles/sample-ora00020.zip --keep
</copy>
```
2. Replace `YOUR_SANDBOX_ID` with that ID to attach. An attached sandbox is preserved.
```bash
<copy>
python -m triage.cli bundles/sample-ora00020.zip --sandbox-id YOUR_SANDBOX_ID
</copy>
```
3. The source also specifies `--snapshot` and `--task`. If you create a snapshot, record it for explicit cleanup. Always finish with the cleanup lab.

Expected result: a reused sandbox avoids creation; neither attachment nor expiration guarantees deletion.

## Acknowledgements

* **Author** - Luigi Saetta
* **Last Updated By/Date** - Luigi Saetta, November 2026
