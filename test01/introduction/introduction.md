# Build a Support Bundle Triage Agent

## About this Workshop

A customer reports intermittent checkout failures and sends a ZIP containing logs, configuration, metrics and a change log. Your task is to build an AI support engineer that investigates that bundle and produces a report you can review: root cause, cited evidence, timeline, confidence and next steps.

By the end you will have four Python modules in your source checkout and a working command:

```bash
<copy>
python -m triage.cli bundles/sample-ora00020.zip --keep
</copy>
```

The command writes `out/sample-ora00020-report.md` and `out/sample-ora00020-report.json`. Success means that the acceptance tests pass, the report explains the incident using verified `file:line` citations, and you have reviewed the findings and cleaned up your sandbox. A citation check proves that the first cited line exists; it does not prove that the model's conclusion is correct.

You build the agent with Codex in three stages after setup: small sandbox tools, a Responses API investigation loop, then structured reporting and a command line interface. Configuration loading, OCI clients, sandbox lifecycle, safe ZIP ingestion and the sample bundle generator are provided. The model selects tools and reasons over their results; the tools execute bundle analysis inside an OCI Sandbox. The local client orchestrates the investigation and writes the reports.

Estimated Time: 5 minutes

### Objectives

* Build and test `triage/tools.py`, `triage/agent.py`, `triage/report.py` and `triage/cli.py`.
* Select venv or Conda, and OCI Generative AI or OpenAI for the model.
* Investigate the supplied case, inspect evidence and manage sandbox ownership.

### Prerequisites

* Git, Codex and a Bash or Zsh terminal on macOS/Linux, or Linux inside WSL.
* Either installed Python 3.11+ (the venv walkthrough uses 3.12), or Conda, which installs Python for you.
* An OCI tenancy, a suitable compartment, project and sandbox permissions, supported service region and available quota.
* OCI CLI and SDK credentials, plus an API key for your selected model provider.
* Internet access to the provider APIs and Oracle's LiveLabs renderer. Model and sandbox usage may incur charges; confirm access, limits and pricing with your administrator.

## Task 1: Choose your route

1. In Lab 0 choose **venv** or **Conda**; both rejoin at dependency installation.
2. Choose **OCI Generative AI** or **OpenAI** for the model. Both use the Responses API with stored conversations. OCI remains required for sandbox execution in either route.
3. Complete Labs 1–3 in order. Run each lab's local acceptance checks before continuing.

| Stage | What you produce | Estimated time |
| --- | --- | --- |
| Lab 0: Setup and preflight | Active Python environment and checked OCI/model configuration | 5 minutes |
| Lab 1: Sandbox tools | Four tools behind `ToolRegistry` | 5 minutes |
| Lab 2: Agent loop | `TriageAgent` with stored response chaining | 5 minutes |
| Lab 3: Report and end-to-end run | Markdown/JSON findings and completed cleanup | 5 minutes |

The four labs are estimated at 20 minutes of guided activity. Installation, Codex iterations and sandbox start/stop waits can extend elapsed time. The source guides allocate about two hours for a full implementation; the five-minute estimates here describe a compressed facilitated walkthrough.

## Task 2: Understand the three services

| Concern | OCI model route | OpenAI model route |
| --- | --- | --- |
| Investigation model | OCI Generative AI, selected by `model.model_id` | OpenAI, selected by `model.model_id` |
| Stored conversation | OCI project; conversation retention and short-term compaction configured there | OpenAI; OCI retention and compaction settings do not apply |
| Execution of bundle commands | OCI Sandbox | OCI Sandbox |

`store=True` and `previous_response_id` continue a server-side conversation. OCI project short-term memory uses its configured condenser; the client does not call it directly. This application does not configure OpenAI compaction or cross-conversation long-term memory. Bundle excerpts returned by tools are sent to the chosen model provider.

Continue with **Lab 0: Setup and preflight** in the navigation menu.

## Acknowledgements

* **Author** - Luigi Saetta
* **Last Updated By/Date** - Luigi Saetta, November 2026
