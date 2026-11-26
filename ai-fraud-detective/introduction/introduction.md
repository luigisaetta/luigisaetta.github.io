# AI Fraud Detective

## Introduction

An anonymous report suggests that supplier payments at Halden Manufacturing may be diverted. Management asks an external forensic firm to investigate. You are one of its analysts, starting with nine months of payment records, emails and control policies. The challenge is to connect evidence across those records without mistaking an unusual transaction for wrongdoing. The company and all case data are invented.

You will explore four ideas:

* **Give AI a way to investigate.** Small tools let a model search records, compare evidence and follow leads across documents.
* **Contain the analysis.** Model-written code runs in an isolated environment, and the original evidence is checked for changes before use.
* **Make conclusions reviewable.** Findings point to their supporting records, with amounts at risk and uncertainty made explicit.
* **Keep people responsible for action.** Investigators assess the evidence and decide what to do next.

Using Codex, you will build an application that investigates the supplied case bundle and produces a forensic report, a version that other programs can read, and a record of its analysis. The report brings together findings, amounts at risk, supporting evidence, explained anomalies and next steps. You will know you have succeeded when you can follow its conclusions back to the records and explain which conclusions the evidence supports.

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

* **Lab 0 — Prepare the environment:** connect the model and the isolated workspace, then check that they are ready.
* **Lab 1 — Give the agent tools:** let it explore records through small, controlled operations.
* **Lab 2 — Build the investigation loop:** let it choose tools, use their results and decide when to stop.
* **Lab 3 — Produce and review the report:** investigate the case, inspect the evidence and release cloud resources.

**Lab 4 is optional.** It adds focused questions about the completed report, with probabilities to support the investigators' next decision. The application is complete after Lab 3.

Expected result: you can describe what your investigator will produce and how you will assess it. Continue with [Lab 0](../workshops/index.html?lab=lab0).

## Acknowledgements

* **Author** - L. Saetta
* **Last Updated By/Date** - L. Saetta, November 2026
