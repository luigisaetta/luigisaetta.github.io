# Lab 1: Build Sandbox Tools

## Introduction

Implement `triage/tools.py` over the provided `SandboxSession`. The four tools expose small pieces of evidence to the model while keeping real bundle commands inside OCI. This lab uses controlled local fakes, with no model or OCI calls.

Estimated Time: 5 minutes

### Objectives

* Implement four strict function tool schemas.
* Bound tool output and execution time.
* Reject unsafe paths and return actionable errors.

### Prerequisites

* Source checkout and installed Python dependencies from Lab 0.
* Read `AGENTS.md`, `labs/lab1.md`, `prompts/lab1.md` and `tests/lab1/test_tools.py`.

## Task 1: Observe the missing module

1. Run the acceptance suite before implementing the module.
```bash
<copy>
python -m pytest tests/lab1 -q
</copy>
```
Expected result at the pinned starting revision: collection fails because `triage.tools` is absent. If you already built it, tests may run; preserve that implementation.

## Task 2: Build the tools with Codex

1. Open Codex at the source repository root. Submit the following instruction as a prompt, not a shell command.
```text
<copy>
Read AGENTS.md and labs/lab1.md. Read prompts/lab1.md and follow the instructions below its opening separator. Build only triage/tools.py. Read the existing acceptance tests first. Do not read solutions/ or modify the provided modules or existing tests.
</copy>
```
2. Review the generated module against this contract:

| Tool | Inputs | Result |
| --- | --- | --- |
| `list_files` | relative `path` | sorted file sizes, at most 200 entries |
| `read_file` | `path`, inclusive `start_line`, `end_line` | numbered evidence, at most 200 lines |
| `grep` | regex `pattern`, `path` | numbered matches, at most 50 |
| `run_python` | `code` | script output from sandbox execution |

3. Verify `Tool.schema()` uses strict JSON schema, required properties and `additionalProperties=False`. Check that `ToolRegistry.call()` accepts JSON text or a dictionary and returns `ERROR:` messages for invalid input rather than raising.
4. Check the 8,000-character cap, head/tail truncation notices and default 60-second timeout. Paths must remain under `/workspace/bundle`; escaping symlinks must be skipped or rejected. Quote shell values with `shlex.quote`. Write analysis scripts to `/workspace/scratch` through the session before running them.

Expected result: the implementation follows the prompt's interfaces and uses the supplied session for all production bundle operations. The fake client is a test harness, not a substitute for isolation when handling untrusted bundles.

The path checks constrain the file-oriented tools. `run_python` executes model-generated code inside the sandbox; these checks do not make it a separate filesystem jail. The OCI Sandbox is the execution boundary, and `/workspace` is its writable workspace. ZIP ingestion separately rejects unsafe archive entries before extraction.

For portability, review shell commands for GNU-only options such as `grep -P` or `find -printf`: the controlled fakes run the same commands on the participant's machine.

## Task 3: Validate the module

1. Run the infrastructure and Lab 1 tests, formatting check and linter.
```bash
<copy>
python -m pytest tests/unit tests/lab1 -q
python -m black --check triage tests
python -m pylint triage
</copy>
```
2. Review `git diff` and confirm that the provided files and existing tests were preserved. Fix implementation failures before continuing.

Expected result after implementation: all selected tests and checks pass. Coverage includes timeouts, shell metacharacters, binary files, symlink escapes and bounded results. These outcomes must be established in your working checkout; they are not assumed.

**Done when:** the cumulative tests pass, provided files are unchanged, and you can explain the difference between a bounded tool result and sandbox isolation. Continue to [Lab 2](../workshops/freetier/index.html?lab=agent-loop).

## Acknowledgements

* **Author** - Luigi Saetta
* **Last Updated By/Date** - Luigi Saetta, November 2026
