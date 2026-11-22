# Build a Support Bundle Triage Agent

## About This Workshop

Diagnosing a support incident often requires connecting evidence from application logs, database alerts, configuration files, and metrics.

In this workshop, you will use **OpenAI Codex** to build an **AI agent** that investigates a **support bundle**: an archive of diagnostic files collected during an incident. The agent will produce a structured **triage report** that includes:

- The likely root cause
- Supporting evidence with verifiable citations
- A timeline of relevant events
- A confidence assessment
- Recommended next steps

### What You Will Learn

You will learn how to give a model tools to explore files, run Python analysis, and correlate evidence across multiple sources. You will also use the **OpenAI Responses API** to manage a multi-step investigation and turn its findings into a structured report.

### How the Agent Works

The model plans the investigation, selects tools, writes Python code to analyze diagnostic data, and reasons over the results. The local client orchestrates these steps and writes the final report.

The model runs through either **OCI Generative AI** or **OpenAI**. In both cases, **OCI Sandbox** provides the isolated execution environment.

Customer diagnostic files are treated as untrusted input. Because the agent can generate and execute analysis code, all analysis runs inside the sandbox with outbound network access disabled. This isolation helps protect your machine and contain code execution.

### What You Will Build

After setup, you will build the agent in three stages:

1. **Sandbox tools** — Implement small tools for exploring and analyzing diagnostic files.
2. **Investigation loop** — Use the Responses API to coordinate tool calls and gather evidence across multiple steps.
3. **Reporting and CLI** — Generate a structured triage report and expose the agent through a command-line interface.

The workshop provides configuration loading, OCI clients, sandbox lifecycle management, safe ZIP ingestion, and a sample bundle generator, so you can focus on the agent's investigation workflow.

### Expected Outcome

By the end of the workshop, you will have implemented four Python modules in your source checkout and will be able to run:

```bash
<copy>
python -m triage.cli bundles/sample-ora00020.zip
</copy>
```

The command investigates the sample bundle and writes the completed report to:

```text
out/sample-ora00020-report.md
```

Estimated Time: 40–60 minutes

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
