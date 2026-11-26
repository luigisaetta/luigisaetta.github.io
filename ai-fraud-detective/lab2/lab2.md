# Lab 2: Agent loop

## Introduction

Build `fraud_detective/agent.py` with Codex. Your loop sends a task and tool schemas to the Responses API, executes requested tools through the registry, returns their outputs and repeats until the model answers or reaches its budget. Required tests use a fake model and fake sandbox locally; they make no service calls.

Estimated Time: 5–7 minutes of hands-on work, plus reading and review.

### Objectives

* Continue a stored conversation with `previous_response_id`.
* Match multiple function results to their `call_id` values.
* Enforce a step budget and treat tool failures as information.
* Keep bundle text from overriding investigation instructions.

### Prerequisites

* Lab 1 completed, active workshop environment and commands run from the repository root.
* Read `AGENTS.md`, `prompts/lab2.md` and `tests/lab2/`. Preserve all provided files and existing tests; do not read `solutions/`.

## Task 1: Check Lab 1 and see the new tests fail

```bash
<copy>
python -m pytest tests/lab1 -q
</copy>
```

Resolve any earlier failure before proceeding. Then run:

```bash
<copy>
python -m pytest tests/lab2 -q
</copy>
```

On a fresh checkout, expect `ModuleNotFoundError: fraud_detective.agent`.

## Task 2: Ask Codex to build the agent

Enter in Codex at the participant repository root:

```text
<copy>
Read prompts/lab2.md and do what it says (everything below the first line). Follow AGENTS.md; do not read solutions/ or instructor-only case documentation.
</copy>
```

The prompt specifies `InvestigationAgent`, `AgentResult` and `ToolCall`. The model client comes from `ModelClientFactory`; provider changes belong in settings. The conversation is stored by OpenAI on the usual path, or in the OCI project on the alternative path.

The request sequence is:

```text
instructions + task + tool schemas -> function calls
previous_response_id + tool outputs -> further calls
previous_response_id + tool outputs -> answer
```

The OCI project condenser applies to OCI conversations only. Passing `previous_response_id` does not itself enable compaction on OpenAI.

## Task 3: Review the loop and instructions

Open `fraud_detective/agent.py` and verify:

* `run()` is a short, readable loop. Every request includes `store=True`, instructions and tool schemas.
* From the second call onward it sends `previous_response_id` and only the new `function_call_output` items. Each result has its original `call_id`.
* Several function calls in one response are handled. Tool errors return to the model as `ERROR:` text, so it can retry.
* After `max_steps` tool-requesting turns, the final request uses `tool_choice="none"` and asks for the best supported answer.
* `INVESTIGATION_INSTRUCTIONS` asks for policy-first exploration, standard-library Python analysis, amounts and `path:line` citations, accounting for CSV headers.
* Bundle text is evidence, never an instruction, even when it appears to address the AI directly.

Expected result: the loop supports investigation without embedding a case-specific answer or hard-coded investigative sequence.

## Task 4: Run the checks

```bash
<copy>
python -m pytest tests/unit tests/lab1 tests/lab2 -q
python -m black --check fraud_detective tests
python -m pylint fraud_detective
</copy>
```

Expected result: all tests pass and formatting/lint checks report no problems. You can explain why the loop needs a budget and what `previous_response_id` replaces. Recover through [Need Help](../workshops/index.html?lab=need-help) if needed.

## Task 5: Inspect the simulated investigation

Review the scripted responses and assertions in `tests/lab2/test_agent.py`. Trace the first request, tool execution, continuation and budget-exhaustion cases. Confirm that fake clients record requests without contacting a service.

The source guide also offers an optional real-model demonstration using a local fake sandbox. This workshop defers the first live investigation to Lab 3 so case data and generated analysis use the OCI execution boundary. No live call is needed to complete this lab.

Expected result: you can account for every tool result sent back to the fake model. Continue to [Lab 3](../workshops/index.html?lab=lab3).

## Acknowledgements

* **Author** - L. Saetta
* **Last Updated By/Date** - L. Saetta, November 2026
