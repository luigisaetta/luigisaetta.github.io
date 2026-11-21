# Lab 3 — Report and end-to-end run

## Introduction

Estimated Time: 5 minutes

**You build:** `triage/report.py` and `triage/cli.py`, with Codex.

**Runs on:** a real OCI Sandbox and the real model.

### Objectives

Put everything together: one command takes a support bundle, runs the
investigation in an OCI Sandbox and writes a triage report.

```
python -m triage.cli bundles/sample-ora00020.zip
```

1. **Sandbox** — create one (or attach to an existing one).
2. **Bundle** — check the zip locally, upload it, extract it in the sandbox.
3. **Investigation** — the agent of Lab 2 explores the bundle with the tools
   of Lab 1.
4. **Report** — a structured report, with the first line of each citation checked in the bundle.
5. **Cleanup** — delete the sandbox, keep it for another run, or snapshot it.

### Why it matters

- **The report is one more turn of the same conversation.** The model already
  holds every tool result on the server: the client sends only
  `previous_response_id` and a request for the report, and nothing is resent.
- **Structured output.** The report follows a strict JSON schema (root cause,
  confidence, evidence, timeline, next steps), so a program can use it, and a
  Markdown version is rendered for people.
- **Evidence you can check.** For each citation such as
  `db/alert_orcl.log:29`, the CLI reads that line in the sandbox and marks it
  ✔ or ✘. A ✘ means the first cited line could not be verified (for example, a missing file, a line beyond EOF or an unparseable location). A ✔ checks existence of the first cited line only, not the entire range or the correctness of the observation.
- **Sandbox lifecycle.** Creating a sandbox takes minutes; `--keep` and
  `--sandbox-id` let you reuse one; `--snapshot` saves the workspace of the
  investigation.

### Prerequisites

Lab 3 builds on your `triage/tools.py` (Lab 1) and `triage/agent.py`
(Lab 2). Check them:

```bash
<copy>
python -m pytest tests/lab1 tests/lab2 -q
</copy>
```

If the tests fail or a file is missing, copy the reference solutions:

```bash
<copy>
cp solutions/lab1/tools.py solutions/lab2/agent.py triage/
</copy>
```

Then write the sample bundle:

```bash
<copy>
python -m triage.make_bundle
</copy>
```

It writes `bundles/sample-ora00020.zip`: logs, configuration, metrics and a
change log of a customer case, with a known root cause, some red herrings and
a line that tries to give orders to the AI assistant.


## Task 1: See the tests fail

```bash
<copy>
python -m pytest tests/lab3 -q
</copy>
```

They fail with `ModuleNotFoundError`: `triage.report` and `triage.cli` do
not exist yet.

## Task 2: Ask Codex to build the report and the CLI

Open Codex in the repository root and enter:

```text
<copy>
Read prompts/lab3.md and do what it says (everything below the first line).
</copy>
```

The tests use a fake model and a fake sandbox: Codex does not start a real
sandbox. You will run the CLI yourself in step 5.

## Task 3: Review what Codex wrote

- `triage/report.py`: `ReportWriter.write()` sends `previous_response_id`,
  `store=True` and the strict JSON schema; `EvidenceChecker` reads the first
  cited line of each evidence and never raises.
- `triage/cli.py`: the five numbered steps; the cleanup runs also after an
  error or Ctrl+C; a sandbox given with `--sandbox-id` is never deleted.

## Task 4: Run the checks

```bash
<copy>
python -m pytest tests/unit tests/lab1 tests/lab2 tests/lab3 -q
python -m black --check triage tests
python -m pylint triage
</copy>
```

## Task 5: Run the triage on a real sandbox

```bash
<copy>
python -m triage.cli bundles/sample-ora00020.zip --keep
</copy>
```

The sandbox start can take several minutes: the CLI shows its state while
you wait. At the end it prints the root cause, the confidence, how many
cited lines were verified, and — because of `--keep` — the id of the
sandbox and the command to reuse it.

Open the report:

```bash
<copy>
cat out/sample-ora00020-report.md
</copy>
```

Check it:

- Does the root cause name the change that lowered the database `processes`
  limit, and the gap between that limit and the connection pools?
- Are all cited lines verified (✔)?
- Did the model ignore the red herrings (disk usage, GC pauses, deprecated
  setting)?
- Did it follow the line addressed to the AI assistant? It must not.

## Task 6: Run again without waiting (optional)

Reuse the sandbox from step 5 — no start-up wait. Replace `YOUR_SANDBOX_ID` with the ID printed by the first run:

```bash
<copy>
python -m triage.cli bundles/sample-ora00020.zip --sandbox-id YOUR_SANDBOX_ID
</copy>
```

Try `--snapshot` to save the workspace, or `--task` to ask a different
question about the same bundle.

## Task 7: Clean up

A sandbox kept with `--keep` expires after `sandbox.lifetime_minutes` (60 by
default) and becomes `STOPPED`, but it is not deleted: delete it from the OCI
Console. Without `--keep`, the CLI deletes the sandbox at the end; if the
service is slow to stop it, the CLI prints a warning and you delete it from
the Console once it is `STOPPED`.

An attached sandbox (`--sandbox-id`) is preserved even without `--keep`; it remains your responsibility. If you request `--snapshot`, record the snapshot ID and remove the workshop-owned snapshot through the OCI Console when no longer needed. Stop and delete only sandboxes you created for this workshop; never delete a shared project. Delete a dedicated workshop project only after checking it has no resources or conversations you need to retain. Stored responses may remain until their provider retention period ends.

Keep reports you need; remove generated sample ZIPs and `out/` files manually only after reviewing them. Leave the source checkout and Python environment available if you plan to continue.

## Task 8: Compare your findings after the investigation

After reading your report, consult `docs/sample-bundle.md` in the source checkout. It identifies CHG-4821, the reduction of `processes` from 1500 to 300, and the application's aggregate pool of 450 connections. Check the timeline and distinguish this causal chain from unrelated disk usage, GC pauses and deprecated settings. Do not treat verified line numbers alone as proof of the conclusion. The injected instruction in the log must remain evidence, never become an instruction to the agent.

## Done when

- `python -m pytest tests/unit tests/lab1 tests/lab2 tests/lab3 -q` passes.
- `out/sample-ora00020-report.md` names the right root cause with verified
  evidence.
- You can explain why the report request sends no history, and what a ✘ in
  the evidence table would mean.

## Stuck?

Copy the reference solution and run step 5:

```bash
<copy>
cp solutions/lab3/report.py solutions/lab3/cli.py triage/
</copy>
```

## Source guide

Use `labs/lab3.md`, `prompts/lab3.md` and `tests/lab3/` in your source checkout for the full interface contract. Reference solutions are an optional participant catch-up path; copying them replaces the named lab files, so preserve any work you need first.

## Acknowledgements

* **Author** - Luigi Saetta
* **Last Updated By/Date** - Luigi Saetta, November 2026
