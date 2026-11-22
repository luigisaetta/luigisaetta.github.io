# Build a Support Bundle Triage Agent

## About this Workshop

Diagnosing a support incident often means piecing together application logs, database alerts, configuration files and metrics. In this workshop, you will use **Codex** to build an AI agent that investigates a **support bundle**, an archive containing these diagnostic files, and produces a triage report with a root cause, cited evidence, a timeline, confidence and next steps.

You will give the model tools to explore files, run Python analysis and correlate evidence across multiple sources. You will use the **OpenAI Responses API** to manage the investigation across multiple steps and turn its findings into a structured report with verifiable citations.

By the end you will have four Python modules in your source checkout and a working command:

```bash
<copy>
python -m triage.cli bundles/sample-ora00020.zip --keep
</copy>
```

The command writes `out/sample-ora00020-report.md` and `out/sample-ora00020-report.json`. Success means that the acceptance tests pass, the report explains the incident using verified `file:line` citations, and you have reviewed the findings and cleaned up your sandbox. A citation check proves that the first cited line exists; it does not prove that the model's conclusion is correct.

You build the agent with Codex in three stages after setup: small sandbox tools, a Responses API investigation loop, then structured reporting and a command line interface. Configuration loading, OCI clients, sandbox lifecycle, safe ZIP ingestion and the sample bundle generator are provided. The model plans the investigation, selects tools, writes Python analysis code and reasons over the results. The local client orchestrates the investigation and writes the reports.

Customer diagnostic files are untrusted input, and the agent can generate code to analyze them. To protect your machine and contain that execution, all bundle analysis runs inside an isolated **OCI Sandbox with outbound network access disabled**. The model runs through **OCI Generative AI** or **OpenAI**; OCI provides the sandbox in either case.

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

Allow about **40–60 minutes** for the workshop, including time to read the guides and review the generated code. The five-minute estimates above describe a compressed facilitated walkthrough. Sandbox creation may occasionally take longer.

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
