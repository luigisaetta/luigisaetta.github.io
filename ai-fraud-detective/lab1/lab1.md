# Lab 1: Sandbox tools

## Introduction

An investigator needs focused ways to search a large body of evidence. You will use Codex to build Python function tools for the OpenAI Responses API: they search records and run analysis in an OCI Sandbox, with limits on execution time and returned text. Local tests use a fake sandbox so you can check the tools before making cloud calls.

Estimated Time: 5–7 minutes of hands-on work, plus reading and code review.

### Objectives

* Implement bounded file listing, line reads, regular-expression search and Python execution.
* Preserve evidence locations and reject unsafe paths or arguments.
* Return actionable errors without crashing the agent.

### Prerequisites

* Lab 0 completed; repository root open and the selected Python environment active.
* Read `AGENTS.md`, `prompts/lab1.md` and the existing acceptance tests.

## Task 1: See the missing module

You will build `fraud_detective/tools.py`. Start with its acceptance tests, which use synthetic fixtures and a local fake sandbox without OCI or model calls:

```bash
<copy>
python -m pytest tests/lab1 -q
</copy>
```

On a fresh participant checkout this fails with `ModuleNotFoundError: fraud_detective.tools`. If the module already exists, review its current state rather than deleting your work.

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

Open `fraud_detective/tools.py`. It should use the provided `SandboxSession` to run commands and move files; its `run()` method takes a timeout. The fake backend in `tests/unit/fakes.py` is for controlled fixtures only; real bundle analysis uses OCI. Check:

* Each tool has its own class, a model-facing `schema()` and a short sandbox command in `run()`.
* `ToolRegistry.call()` accepts the documented arguments and converts failures to a single short `ERROR: ...` result.
* Paths outside the bundle and symbolic links leaving it are rejected. Model-supplied shell values pass through `shlex.quote`.
* `run_python` writes a script under `/workspace/scratch` and executes it through the session. Every command has a timeout.
* Outputs have limits and report truncation; long results keep useful head and tail portions.

Expected result: each tool returns a bounded answer or an actionable error through the provided sandbox interface.

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
