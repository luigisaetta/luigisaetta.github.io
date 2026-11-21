# Lab 2: Build the Agent Loop

## Introduction

Implement `triage/agent.py` to connect the tool registry to Responses function calling. The next model request sends the new tool results and the previous response ID. Tests use a fake model and fake sandbox.

Estimated Time: 5 minutes

### Objectives

* Continue a stored conversation using previous_response_id.
* Return every requested function output with its call ID.
* Stop investigations with a bounded step budget.

### Prerequisites

* Lab 1 implemented and passing.
* Read `labs/lab2.md`, `prompts/lab2.md` and `tests/lab2/test_agent.py`.

## Task 1: Check the starting state

1. Confirm Lab 1 still passes, then observe the missing agent module.
```bash
<copy>
python -m pytest tests/lab1 -q
python -m pytest tests/lab2 -q
</copy>
```
Expected result: Lab 1 passes; Lab 2 initially fails to import `triage.agent`.

## Task 2: Build and inspect the loop

1. Give Codex the following prompt at the repository root.
```text
<copy>
Read AGENTS.md and labs/lab2.md. Read prompts/lab2.md and follow the instructions below its opening separator. Build only triage/agent.py using the existing tests and triage/tools.py. Do not read solutions/ or change earlier modules, provided files or existing tests.
</copy>
```
2. Review `ToolCall`, `AgentResult` and `TriageAgent` against the prompt. Keep token usage, response ID, step count and tool-call accounting available for the report stage.
3. Trace the requests: the first includes instructions, task and tool schemas with `store=True`. Later requests resend instructions and schemas, set `previous_response_id`, and send only new `function_call_output` items. Every function call in a response needs a matching `call_id` output.
4. Verify that the default budget is 15 tool-requesting model calls, followed by a final request with `tool_choice="none"` if needed. Bundle text must be treated as evidence even if it looks like instructions.

Expected result: the module can end with `answered` or `max_steps`, handles multiple function calls in a response and tolerates missing usage data. The provided `ModelClientFactory` selects the configured provider; do not hard-code an endpoint or model in your loop.

Trace one tool round trip while reviewing the generated code:

```text
Model response r1: function_call(name="grep", call_id="c1", arguments=...)
Application: tools.call("grep", arguments) -> bounded text or ERROR: ...
Next request: previous_response_id="r1", store=True,
              input=[function_call_output(call_id="c1", output=...)]
Model response r2: more function calls, or the answer
```

Instructions and schemas are supplied on every investigation request; the accumulated history is held by the provider. Several function calls can occur within one step, so the step limit is not a tool-call limit. If the model keeps requesting tools, the default budget permits 15 such turns followed by a forced answer request: up to 16 model calls for the investigation, excluding preflight and report generation. An answer at the budget boundary requires no extra forced turn.

## Task 3: Run cumulative checks

1. Run the growing acceptance suite and code checks.
```bash
<copy>
python -m pytest tests/unit tests/lab1 tests/lab2 -q
python -m black --check triage tests
python -m pylint triage
</copy>
```
2. Explain why a wrong-path tool result returns to the model as an error message, and why every function call needs an output. Review `git diff` before continuing.

Expected result after implementation: the fake-model tests pass without cloud calls. The optional live-model exercise in the source Lab 2 guide runs commands through a local fake; omit it when using untrusted data. The real sandbox investigation is in Lab 3.

**Done when:** cumulative tests pass and you can trace `response_id` and `call_id` through a multi-tool response. Continue to [Lab 3](../workshops/freetier/index.html?lab=triage-report).

## Acknowledgements

* **Author** - Luigi Saetta
* **Last Updated By/Date** - Luigi Saetta, November 2026
