# Introduction

## Introduction

Build a support bundle triage agent with Codex. The agent investigates synthetic database and application logs, connects evidence across files, and writes a structured report. You build four Python modules while retaining the supplied OCI infrastructure and acceptance tests.

The production path is: CLI → safe ZIP inspection and sandbox upload → agent loop → function tools → OCI Sandbox → structured report and citation checks. The model uses the Responses API through OCI Generative AI or OpenAI. OCI is required for sandboxes with either model provider.

This workshop follows source revision `228ad6756adcc41395ac76e86f0a2de48735dbf8`. That revision contains infrastructure, prompts and tests; `triage/tools.py`, `triage/agent.py`, `triage/report.py` and `triage/cli.py` are intentionally absent. The README's statement that implementations are included does not match that revision. Do the labs in sequence.

Allow about 2 hours for setup and the three build labs, plus introduction, cleanup and cloud startup waits. This workshop has been reviewed against the source; live OCI provisioning, model calls and the completed agent run have not been executed for this edition.

Estimated Time: 10 minutes

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

## Task 1: Choose your learning path

1. Complete **Lab 0: Setup and Preflight** for the cloud path. Labs 1 and 2 and the build portion of Lab 3 use offline fakes after dependency installation; you can study those before obtaining cloud access.
2. Build tools, then the agent, then the report and CLI.
3. Run the synthetic investigation only after preflight succeeds, then complete cleanup.

Expected result: you know which steps use cloud services. Quick preflight calls the model and OCI APIs; full preflight and the final CLI run also create sandboxes. Dependency installation needs internet access. The fake sandbox executes test commands locally and is only for the controlled fixtures supplied by this repository. Never use it with customer bundles.

## Task 2: Plan access and resource use

1. Ask your OCI administrator to confirm project, sandbox, model and compaction-model availability in the selected region and permissions to create, inspect, stop and delete workshop resources.
2. Check applicable service pricing and quotas before the cloud steps. Model calls, sandbox runtime and retained resources may incur costs; the URL folder named `freetier` does not guarantee a free workshop.
3. Use synthetic data only. OCI project configuration retains responses and conversations for 720 hours; the OpenAI provider uses its own account and retention controls.

Read [Oracle project creation guidance](https://docs.oracle.com/en-us/iaas/Content/generative-ai/create-project.htm) and the [source project](https://github.com/luigisaetta/support-bundle-triage-agent). Exact IAM policies and account-specific availability are prerequisites to resolve with your administrator. Expected result: an agreed region, permissions and resource budget.

## Acknowledgements

* **Author** - Luigi Saetta
* **Last Updated By/Date** - Luigi Saetta, November 2026
