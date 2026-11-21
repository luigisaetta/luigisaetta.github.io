# Lab 2 — Agent loop

## Introduction

Estimated Time: 5 minutes

**You build:** `triage/agent.py`, with Codex.

**Runs on:** your machine, with a fake sandbox and a scripted model client. No live model calls are needed in this lab.

### Objectives

In Lab 1 you gave the model tools. Now you build the **agent**: the loop that
lets the model investigate a bundle on its own. It sends the task to the
model, runs the tools the model asks for in the sandbox, sends back the
results, and repeats until the model answers.

The loop uses the **server-side conversation** of the Responses API
(`store=True`): the selected model provider keeps the whole conversation, and each call only sends
what is new.

```
call 1:  instructions + task + tool schemas                 -> function calls
call 2:  previous_response_id + tool outputs                -> function calls
...
call N:  previous_response_id + tool outputs                -> answer
```

### Why it matters

- **Server-side conversation.** The client does not resend the history: it
  points to the previous response with `previous_response_id`. Requests stay
  small, and the conversation can be continued later from its id (Lab 3 does
  it to ask for the report). On the OCI model route, project short-term memory condenses long conversations; OCI settings do not apply to the OpenAI model route.
- **Several tools per step.** The model can ask for several tools in one
  response (for example, read three files at once). Each output goes back
  with the `call_id` of its call.
- **A step budget.** An investigation must end. After `max_steps` calls that
  request tools, the agent asks for the best answer with the evidence it has.
- **Tool errors are information.** A wrong path or a timeout goes back to the
  model as an `ERROR:` text; the model corrects itself and the loop goes on.
- **Prompt injection.** The bundle comes from a customer system and can
  contain text written for the model ("ignore previous instructions..."). The
  instructions of the agent say that bundle content is evidence, never an
  instruction.

### Prerequisites

Lab 2 builds on your `triage/tools.py` from Lab 1. Check it:

```bash
<copy>
python -m pytest tests/lab1 -q
</copy>
```

If the tests fail or the file is missing, copy the reference solution:

```bash
<copy>
cp solutions/lab1/tools.py triage/tools.py
</copy>
```


## Task 1: See the tests fail

```bash
<copy>
python -m pytest tests/lab2 -q
</copy>
```

In a clean starter, expect `ModuleNotFoundError: triage.agent`. If the module already exists, use the run as a baseline and review its implementation.

## Task 2: Ask Codex to build the agent

Open Codex in the repository root and enter:

```text
<copy>
Read prompts/lab2.md and do what it says (everything below the first line).
</copy>
```

The prompt explains the loop as verified on OCI, the interface
(`TriageAgent`, `AgentResult`, `ToolCall`), the instructions the agent must
give the model, and the tests to write. The tests use a fake model client:
no model call is made.

## Task 3: Review what Codex wrote

Open `triage/agent.py` and check:

- `run()` is a short, readable loop.
- Every request has `store=True`, the instructions and the tool schemas;
  from the second one on, `previous_response_id` and only the new
  `function_call_output` items.
- When the budget runs out, the last call has `tool_choice="none"`.
- `TRIAGE_INSTRUCTIONS` asks for `path:line` evidence and treats bundle
  content as untrusted data.

## Task 4: Run the checks

```bash
<copy>
python -m pytest tests/unit tests/lab1 tests/lab2 -q
python -m black --check triage tests
python -m pylint triage
</copy>
```

## Task 5: Review the investigation boundary

The source guide offers an optional live-model demonstration over `FakeSandboxClient`. That fake executes commands locally and is intended for controlled tests; it is deliberately omitted here for live model-generated code. Use scripted fakes for acceptance tests and wait for Lab 3 to run a live investigation in a real OCI Sandbox.

Expected result: the tests demonstrate response chaining, multiple tools per response, recoverable tool errors and the final budget-limited answer without any live model or OCI calls.

## Done when

- `python -m pytest tests/unit tests/lab1 tests/lab2 -q` passes.
- You can explain what `previous_response_id` replaces, and why the agent
  needs a step budget.

## Stuck?

Copy the reference solution and go on to Lab 3:

```bash
<copy>
cp solutions/lab2/agent.py triage/agent.py
</copy>
```

## Source guide

Use `labs/lab2.md`, `prompts/lab2.md` and `tests/lab2/` in your source checkout for the full interface contract. Reference solutions are an optional participant catch-up path; copying them replaces the named lab files, so preserve any work you need first.

## Acknowledgements

* **Author** - Luigi Saetta
* **Last Updated By/Date** - Luigi Saetta, November 2026
