# AI Fraud Detective

## Introduction

Halden Manufacturing, a fictitious industrial company, has received an anonymous report that supplier payments may be diverted. You are an analyst at the external forensic firm engaged to investigate. You start with a case bundle containing nine months of accounts payable exports, change logs, emails, employee records, control policies and case files. All people, companies and bank details are invented.

You will use Codex to build an agent that turns that bundle into a forensic report with findings, amounts at risk in EUR, confidence, cited evidence, cleared anomalies, a timeline and next steps. Your finished application runs from the project root:

```bash
<copy>
python -m fraud_detective.cli bundles/halden-ap-q3-2026.zip --keep
</copy>
```

The result is a Markdown report, a machine-readable JSON report and a trace of every tool call under `out/`. Success means you can trace the report's conclusions back to the supplied evidence and explain its limitations. A citation check confirms that a cited line exists; investigators must still decide whether that line supports the conclusion.

Lab 0 prepares and checks the environment. In Lab 1 you build bounded file and Python tools; in Lab 2 you build the investigation loop; in Lab 3 you add the structured report and command line, investigate the case and clean up. Lab 4 is optional decision support on the saved report. Configuration, OCI clients, sandbox management, integrity checks, sample generation and acceptance tests are provided.

Estimated Time: 40–60 minutes for Labs 0–3 including reading and code review; allow another 5–7 minutes of hands-on work plus review for optional Lab 4. Account setup and slow sandbox provisioning can add time.

### Objectives

* Build and review a tool-calling investigation agent with Codex.
* Verify bundle integrity before analysis and execute analysis in OCI Sandboxes.
* Continue a Responses API conversation with `store=True` and `previous_response_id`.
* Produce and review structured findings, citations and an execution trace.
* Manage sandbox ownership and cleanup; optionally ask bounded decision questions.

### Prerequisites

* Git, Codex and a Bash or Zsh terminal on macOS or Linux; on Windows, use WSL.
* Either Conda (which installs Python) or Python 3.12 for the venv path. The source package requires Python 3.11 or later; this workshop uses 3.12.
* An OCI tenancy, sandbox-capable region, compartment, OCI CLI and configured OCI credentials. Ask the administrator to review the [source IAM and capacity guide](https://github.com/luigisaetta/ai-fraud-detective/blob/f6ee1a8b896f0369dc235e4cec1e94012c4b54eb/docs/oci-policies.md).
* An OpenAI API key and model access for the usual path, or an OCI Generative AI API key and regional model access for the alternative. Lab 4 normally uses OpenAI.
* Budget and quota for model calls and OCI resources. This workshop does not claim free-tier eligibility.

## Task 1: Understand the execution and data flow

Codex writes the missing application modules from repository prompts and tests. At runtime the selected model plans the investigation, requests tools and writes analysis code. Tool commands execute in an OCI Sandbox under `/workspace`, with timeouts, bounded outputs and outbound networking disabled. Local unit tests use a fake sandbox with synthetic fixtures; they do not provide production isolation.

`DENY_ALL` blocks outbound connections from sandbox code. Your local application still retrieves tool output and sends it to the model provider; reports and traces are also written locally. Use the invented workshop data. The hash establishes integrity against the reference extraction, not the truth of individual records. Bundle text is untrusted evidence and cannot override agent instructions.

| Layer | OpenAI, usual path | OCI Generative AI, alternative |
| --- | --- | --- |
| Investigation model | OpenAI Responses API | OCI OpenAI-compatible Responses API |
| Stored conversation | OpenAI, according to platform/account settings | OCI project, configured for 720-hour retention |
| Compaction | No explicit OpenAI compaction configured by this application | OCI project short-term memory with its configured condenser |
| Code execution | OCI Sandbox | OCI Sandbox |
| Optional Lab 4 | OpenAI Decisions API | OpenAI by default; OCI alternative where available |

Neither path enables application long-term memory across investigations. OCI project conversation settings do not configure OpenAI conversation storage.

Expected result: you can identify which service receives model requests and which service executes analysis code. Continue with [Lab 0](../workshops/index.html?lab=lab0).

## Acknowledgements

* **Author** - L. Saetta
* **Last Updated By/Date** - L. Saetta, November 2026
