# Lab 1: Sandbox tools

## Introduction

Build `fraud_detective/tools.py` with Codex. The agent will explore a bundle through four small function tools rather than loading every record into model context. This lab runs locally against a fake sandbox with synthetic test fixtures and makes no OCI or model calls.

Estimated Time: 5–7 minutes of hands-on work, plus reading and code review.

### Objectives

* Implement bounded file listing, line reads, regular-expression search and Python execution.
* Preserve evidence locations and reject unsafe paths or arguments.
* Return actionable errors without crashing the agent.

### Prerequisites

* Lab 0 completed; repository root open and the selected Python environment active.
* Read `AGENTS.md`, `prompts/lab1.md` and the existing acceptance tests. Keep provided modules and tests unchanged; do not read `solutions/`.

## Task 1: See the missing module

```bash
<copy>
python -m pytest tests/lab1 -q
</copy>
```

On a fresh participant checkout this fails with `ModuleNotFoundError: fraud_detective.tools`. If the module already exists, review its current state rather than deleting your work.

The provided `SandboxSession` has `run(command, timeout_seconds)`, `write_file(path, data)` and `read_file(path)`. `tests/unit/fakes.py` supplies the fake backend. Its local execution is for controlled test fixtures only; real bundle analysis must use OCI.

## Task 2: Ask Codex to build the tools

Open Codex in the participant repository root and enter this prompt:

```text
<copy>
Read prompts/lab1.md and do what it says (everything below the first line). Follow AGENTS.md; do not read solutions/ or instructor-only case documentation.
</copy>
```

The prompt defines `Tool`, the tool classes and `ToolRegistry`. Keep those interfaces because later labs depend on them.

| Tool | Expected output |
| --- | --- |
| `list_files` | Bundle tree and file sizes |
| `read_file` | Requested line range |
| `grep` | Matching lines with file locations |
| `run_python` | Bounded output of an analysis script |

Evidence uses `path:line: text`. For CSV files, line 1 is the header and line n is data row n minus 1. Large exports require aggregation through standard-library Python rather than reading every row into context.

## Task 3: Review the generated implementation

Open `fraud_detective/tools.py` and check:

* Each tool has its own class, a model-facing `schema()` and a short sandbox command in `run()`.
* `ToolRegistry.call()` accepts the documented arguments and converts failures to a single short `ERROR: ...` result.
* Paths outside the bundle and symbolic links leaving it are rejected. Model-supplied shell values pass through `shlex.quote`.
* `run_python` writes a script under `/workspace/scratch` and executes it through the session. Every command has a timeout.
* Outputs have limits and report truncation; long results keep useful head and tail portions.

Bundle content and model arguments are untrusted. Production analysis stays inside the sandbox; outbound isolation does not prevent the application from forwarding selected tool output to the model provider.

## Task 4: Run the checks

Run each command and resolve failures in your lab code:

```bash
<copy>
python -m pytest tests/unit tests/lab1 -q
python -m black --check fraud_detective tests
python -m pylint fraud_detective
</copy>
```

Expected result: all tests pass, Black reports no required changes and Pylint reports no problems. Do not edit, skip or remove existing tests to achieve this result.

## Task 5: Try the tools by hand (optional)

In a Python session launched from the repository root, use this synthetic fixture:

```python
<copy>
import tempfile
from pathlib import Path
from tests.unit.fakes import FakeSandboxClient
from fraud_detective.sandbox import SandboxSession
from fraud_detective.tools import ToolRegistry

session = SandboxSession(FakeSandboxClient(Path(tempfile.mkdtemp())), "sbx-fake")
session.write_file("/workspace/bundle/ap/payments.csv", "id,vendor,amount\nP-1,V-17,1250.00\n")
tools = ToolRegistry(session)
print(tools.call("list_files", {"path": "."}))
print(tools.call("grep", '{"pattern": "V-[0-9]+", "path": "."}'))
print(tools.call("read_file", {"path": "../../etc/passwd", "start_line": 1, "end_line": 3}))
</copy>
```

Expected result: a listing, a matching evidence line and an `ERROR:` for the outside path. The JSON string in the second call matches how model arguments arrive in Lab 2.

You are done when the checks pass and you can explain each tool's bounds. If blocked, use [Need Help](../workshops/index.html?lab=need-help), then continue to [Lab 2](../workshops/index.html?lab=lab2).

## Acknowledgements

* **Author** - L. Saetta
* **Last Updated By/Date** - L. Saetta, November 2026
