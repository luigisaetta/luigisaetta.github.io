# Introduction

## Introduction

Build a support bundle triage agent with Codex. The agent investigates synthetic database and application logs, connects evidence across files, and writes a structured report. You build four Python modules while retaining the supplied OCI infrastructure and acceptance tests.

The production path is: CLI → safe ZIP inspection and sandbox upload → agent loop → function tools → OCI Sandbox → structured report and citation checks. The model uses the Responses API through OCI Generative AI or OpenAI. OCI is required for sandboxes with either model provider.

At the reviewed starting revision, the infrastructure, prompts and tests are provided; you create `triage/tools.py`, `triage/agent.py`, `triage/report.py` and `triage/cli.py` in Labs 1–3. Follow the labs in sequence. If you reuse a checkout containing your implementations, review and test them instead of expecting missing-module errors.

Plan for **up to 5 minutes per lab**: about **20 minutes for Labs 0–3**, or **30 minutes including introduction and cleanup**. Need Help is an optional 5-minute reference. These estimates reflect the workshop author’s test on **November 21, 2026**, in which each lab completed in 5 minutes or less. Have the listed prerequisites ready; first-time account preparation, dependency downloads, troubleshooting and cloud lifecycle waits can add time. The timing observation is author-reported; this documentation review did not repeat the live cloud run.

Estimated Time: 5 minutes

### Objectives

* Build bounded tools that inspect files in an OCI Sandbox.
* Continue a stored Responses API conversation and enforce a step budget.
* Generate JSON and Markdown findings and assess cited evidence.
* Track and clean up workshop cloud resources.

### Prerequisites

* Python 3.11 or later; the setup commands use an installed Python 3.12.
* Git, Codex and Bash or Zsh on macOS/Linux, or WSL.
* OCI tenancy, compartment permissions, configured OCI SDK credentials and OCI CLI.
* Access to a model supporting Responses function calling and strict structured output.

### Workshop at a glance

| Section | Deliverable / completion check | Execution | Estimated time |
| --- | --- | --- | --- |
| Introduction | Understand the architecture and choose a path | Reading | 5 min |
| Lab 0 | Environment, configuration and preflight ready | Local + OCI + model | 5 min |
| Lab 1 | `triage/tools.py`; unit and Lab 1 tests pass | Local fakes | 5 min |
| Lab 2 | `triage/agent.py`; cumulative tests pass | Local fakes | 5 min |
| Lab 3 | Report + CLI; tests and synthetic investigation reviewed | Local fakes, then OCI + model | 5 min |
| Cleanup | Workshop resources accounted for | Local + OCI Console | 5 min |

### Two uses of AI

**Codex builds the application** from the prompts and acceptance tests. **The configured runtime model investigates the bundle**: it chooses files and tools, writes Python analysis, correlates timestamps and decides when it has enough evidence. The Python application supplies the loop, isolation, limits and validation; it does not contain a fixed diagnostic recipe.

```text
Participant + Codex -> tools.py -> agent.py -> report.py + cli.py

CLI -> inspect ZIP metadata locally -> upload and extract in OCI Sandbox
    -> model requests tools -> application executes tools in the sandbox
    -> tool outputs continue the stored conversation -> structured report
    -> check citations -> write local JSON + Markdown -> cleanup
```

With the OCI provider, project settings enable short-term memory and a platform-managed compaction model. With OpenAI, conversation storage follows that provider's account controls. The client continues with `previous_response_id`; it never calls the compaction model itself. Long-term memory across conversations is not enabled by this workshop.

Choose an available model that supports reliable strict function calling, structured output and reasoning across files. Correct JSON and real line numbers alone do not establish a correct diagnosis. See the source note [How AI is used](https://github.com/luigisaetta/support-bundle-triage-agent/blob/228ad6756adcc41395ac76e86f0a2de48735dbf8/docs/how-ai-is-used.md) for the design rationale.

## Task 1: Choose your learning path

1. For the full path, complete [Lab 0](../workshops/freetier/index.html?lab=setup), then [Lab 1](../workshops/freetier/index.html?lab=sandbox-tools), [Lab 2](../workshops/freetier/index.html?lab=agent-loop) and [Lab 3](../workshops/freetier/index.html?lab=triage-report). For offline implementation practice, complete Lab 0 Tasks 1–2, then the build and test tasks; return to the cloud setup before the live investigation.
2. Build tools, then the agent, then the report and CLI.
3. Run the synthetic investigation only after preflight succeeds, then complete [cleanup](../workshops/freetier/index.html?lab=cleanup).

Expected result: you know which steps use cloud services. Quick preflight calls the model and OCI APIs; full preflight and the final CLI run also create sandboxes. Dependency installation needs internet access. The fake sandbox executes test commands locally and is only for the controlled fixtures supplied by this repository. Never use it with customer bundles.

## Task 2: Plan access and resource use

1. Ask your OCI administrator to confirm project, sandbox, model and compaction-model availability in the selected region and permissions to create, inspect, stop and delete workshop resources.
2. Check applicable service pricing and quotas before the cloud steps. Model calls, sandbox runtime and retained resources may incur costs; the URL folder named `freetier` does not guarantee a free workshop.
3. Use synthetic data only. OCI project configuration retains responses and conversations for 720 hours; the OpenAI provider uses its own account and retention controls.

Read [Oracle project creation guidance](https://docs.oracle.com/en-us/iaas/Content/generative-ai/create-project.htm) and the [source project](https://github.com/luigisaetta/support-bundle-triage-agent). Exact IAM policies and account-specific availability are prerequisites to resolve with your administrator. Expected result: an agreed region, permissions and resource budget.

## Acknowledgements

* **Author** - Luigi Saetta
* **Last Updated By/Date** - Luigi Saetta, November 2026
