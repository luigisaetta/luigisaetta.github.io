# AI Fraud Detective

## Introduction

An anonymous report suggests that supplier payments at Halden Manufacturing may be diverted. Management asks an external forensic firm to investigate. You are one of its analysts, starting with nine months of payment records, emails and control policies. The challenge is to connect evidence across those records without mistaking an unusual transaction for wrongdoing. The company and all case data are invented.

You will explore five ideas and the technologies behind them:

* **Build with Codex.** Codex helps you write and review the Python application from the lab prompts and tests.
* **Investigate with the OpenAI Responses API.** Its function calling lets the agent request tools and carry evidence forward through a multi-step investigation, using OpenAI as the usual model provider or OCI Generative AI as the alternative.
* **Contain analysis with OCI Sandboxes.** They execute the agent's Python analysis with outbound network access disabled, keeping generated code away from your local system.
* **Make evidence reviewable.** SHA-256 checks detect changes to the case archive, and the OpenAI Responses API produces a structured report whose citations investigators can check.
* **Support decisions with the OpenAI Decisions API.** Optional Lab 4 asks focused questions about the saved report and returns probabilities; investigators decide what action to take.

Your application will investigate the supplied case bundle and produce a forensic report in Markdown and JSON, plus a trace of its analysis. The report brings together findings, amounts at risk, supporting evidence, explained anomalies and next steps. You will know you have succeeded when you can follow its conclusions back to the records and explain which conclusions the evidence supports.

The case data, cloud connections and testing framework are provided. You build the investigation tools and the agent that uses them, then turn its work into a report you can review.

Estimated Time: 40–60 minutes for Labs 0–3 including reading and code review; allow another 5–7 minutes of hands-on work plus review for optional Lab 4. Account setup and slow sandbox provisioning can add time.

### Objectives

* Build an AI investigator with Codex and review its work.
* Protect the evidence and contain the code used to analyse it.
* Assess findings through citations, uncertainty and a record of the analysis.
* Optionally turn a completed report into focused decision-support questions.

### Prerequisites

* Git, Codex and a Bash or Zsh terminal on macOS or Linux; on Windows, use WSL.
* Conda or an installed Python interpreter; Lab 0 walks through both environment choices.
* An OCI account with permission and capacity to use sandboxes, plus the OCI command-line tools and credentials.
* Model access through OpenAI, the usual path, or OCI Generative AI, the alternative. OCI provides the sandbox in both cases.
* Budget for model calls and cloud resources.

## Task 1: Follow the investigation journey

Work through Labs 0–3 in order:

* **Lab 0 — Prepare the environment:** set up Python, model access and OCI Sandboxes, then check that they are ready.
* **Lab 1 — Give the agent tools:** use Codex to build Python tools that explore records in an OCI Sandbox.
* **Lab 2 — Build the investigation loop:** use the OpenAI Responses API to choose tools, use their results and decide when to stop.
* **Lab 3 — Produce and review the report:** investigate the case in an OCI Sandbox, review the report and release cloud resources.

**Lab 4 is optional.** It uses the OpenAI Decisions API to support the investigators' next decision. The application is complete after Lab 3.

Expected result: you can describe what your investigator will produce and how you will assess it. Continue with [Lab 0](../workshops/index.html?lab=lab0).

## Acknowledgements

* **Author** - L. Saetta
* **Last Updated By/Date** - L. Saetta, November 2026
