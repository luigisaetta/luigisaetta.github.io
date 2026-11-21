# Lab 1 — Sandbox tools

## Introduction

Estimated Time: 5 minutes

**You build:** `triage/tools.py`, with Codex.

**Runs on:** your machine, against a fake sandbox. No OCI calls.

### Objectives

During a real investigation, bundle commands run in OCI Sandbox and the model never sees the whole bundle at once. Local acceptance tests use controlled fixtures and a fake sandbox. It explores the bundle by calling a few small **function
tools**, and every tool runs inside the OCI Sandbox. In this lab you build
those tools:

| Tool | What it gives the model |
|------|-------------------------|
| `list_files` | the bundle tree, with file sizes |
| `read_file` | a range of lines of one file |
| `grep` | the lines matching a regular expression |
| `run_python` | the output of a short analysis script |

Every result that points into the bundle uses the format `path:line: text`.
In Lab 3 the triage report cites its evidence in the same format.

### Prerequisites

* Complete Lab 0 and keep your chosen venv or Conda environment active.
* Work from the source repository root.

### Why it matters

- **Isolation.** A support bundle comes from a customer system: it is
  untrusted data. Commands on it run in the sandbox, not on your laptop, and
  the sandbox has no outbound network.
- **Bounded output.** A log can be gigabytes; the model context cannot.
  Each tool returns at most a few thousand characters, and says what it cut.
- **Errors the model can use.** A tool never crashes the agent: a wrong path
  or a timeout comes back as a short `ERROR:` message that tells the model how
  to retry.
- **Untrusted input from the model, too.** Paths and patterns come from the
  model, which reads the bundle. The tools reject paths outside the bundle,
  never follow symbolic links that leave it, and never let a pattern become a
  shell command.

### What is provided

- `triage/sandbox.py` — `SandboxSession`: `run(command, timeout_seconds)`,
  `write_file(path, data)`, `read_file(path)`. You use it; you do not change it.
- `tests/unit/fakes.py` — `FakeSandboxClient`: runs the sandbox commands on
  your machine in a temporary directory, so the tests are fast and free.
- `tests/lab1/` — the acceptance tests of this lab.
- `AGENTS.md` — the rules Codex follows in this repository.


## Task 1: See the tests fail

```bash
<copy>
python -m pytest tests/lab1 -q
</copy>
```

In a clean starter the tests fail with `ModuleNotFoundError: triage.tools`. If your checkout already contains the module, review it and use the tests as a baseline; do not delete working code to force a failure.

## Task 2: Ask Codex to build the tools

Open Codex in the repository root and enter:

```text
<copy>
Read prompts/lab1.md and do what it says (everything below the first line).
</copy>
```

The prompt gives Codex the interface (`Tool`, the four tools,
`ToolRegistry`), the output format with examples, the security rules and the
tests to pass. Codex writes `triage/tools.py`, runs the tests, black and
pylint, and reports.

## Task 3: Review what Codex wrote

Open `triage/tools.py` and check:

- Each tool has its own class, with a `schema()` for the model and a `run()`
  that builds one short command for the sandbox.
- `ToolRegistry.call()` catches every error and returns `ERROR: ...`.
- Values from the model reach the shell only through `shlex.quote`;
  `run_python` writes the code to a file in `/workspace/scratch` and runs it.
- Every command has a timeout.

## Task 4: Run the checks

```bash
<copy>
python -m pytest tests/unit tests/lab1 -q
python -m black --check triage tests
python -m pylint triage
</copy>
```

Expected result: all tests pass, black reports no formatting changes and pylint reports no problems.

## Task 5: Try the tools by hand (optional)

```python
<copy>
import tempfile
from pathlib import Path

from tests.unit.fakes import FakeSandboxClient
from triage.sandbox import SandboxSession
from triage.tools import ToolRegistry

session = SandboxSession(FakeSandboxClient(Path(tempfile.mkdtemp())), "sbx-fake")
session.write_file("/workspace/bundle/logs/app.log", "start\nORA-04031 shared pool\nend\n")
tools = ToolRegistry(session)

print(tools.call("list_files", {"path": "."}))
print(tools.call("grep", '{"pattern": "ORA-[0-9]+", "path": "."}'))
print(tools.call("read_file", {"path": "../../etc/passwd", "start_line": 1, "end_line": 3}))
</copy>
```

The second call passes the arguments as a JSON string, as the model will do in
Lab 2. The third one shows the error the model receives for a path outside the
bundle.

## Done when

- `python -m pytest tests/unit tests/lab1 -q` passes.
- You can explain what each tool returns and why its output is bounded.

## Stuck?

Copy the reference solution and go on to Lab 2:

```bash
<copy>
cp solutions/lab1/tools.py triage/tools.py
</copy>
```

## Source guide

Use `labs/lab1.md`, `prompts/lab1.md` and `tests/lab1/` in your source checkout for the full interface contract. Reference solutions are an optional participant catch-up path; copying them replaces the named lab files, so preserve any work you need first.

## Acknowledgements

* **Author** - Luigi Saetta
* **Last Updated By/Date** - Luigi Saetta, November 2026
