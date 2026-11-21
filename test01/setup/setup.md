# Lab 0: Setup and Preflight

## Introduction

Prepare an isolated Python environment, configure an OCI Generative AI project and verify the model and sandbox. Follow this setup once; on a prepared machine, verify or reuse the existing environment and correctly configured project. The 5-minute estimate uses the author-reported test on November 21, 2026; first-time provisioning, downloads and cloud waits can extend it. Commands are grounded in the reviewed source; this review did not rerun cloud provisioning.

Estimated Time: 5 minutes

### Objectives

* Install the pinned application dependencies.
* Separate local secrets from versioned configuration.
* Distinguish quick preflight from full sandbox validation.

### Prerequisites

* Introduction completed.
* Python 3.12, Git, Codex and the OCI CLI installed.
* OCI SDK credentials configured privately; required tenancy access and API key ready.

## Task 1: Get the reviewed source

1. For a new exercise, clone into a new working directory and select the reviewed revision. If you already have a prepared checkout, confirm its revision and preserve your work; skip the clone and checkout commands rather than changing a working exercise.
```bash
<copy>
git clone https://github.com/luigisaetta/support-bundle-triage-agent.git
cd support-bundle-triage-agent
git checkout --detach 228ad6756adcc41395ac76e86f0a2de48735dbf8
</copy>
```
2. Read `AGENTS.md` and `labs/lab0.md`. Keep the provided modules, tests and reference solutions unchanged. Ask Codex to work only on the current lab modules.

Expected result: a checkout at `228ad6756adcc41395ac76e86f0a2de48735dbf8`, with four modules to build in a fresh clone. This fixed revision keeps the prompts and tests consistent. The detached checkout is suitable for local exercises; preserve your work before changing revisions. The source README says implementations are included, but those four files are absent at this revision; the lab tests and prompts define what you build.

## Task 2: Create the environment

1. Confirm the interpreter, create and activate the environment, then install dependencies. Run each line separately.
```bash
<copy>
python3.12 --version
python3.12 -m venv .venv
source .venv/bin/activate
python -c "import sys; print(sys.executable)"
python -m pip install -r requirements.txt -r requirements-dev.txt
oci --version
</copy>
```
2. Use `python` for the remaining lab commands. Activate this environment again in every new terminal.

Expected result: Python resolves inside `.venv`; package installation completes and the separately installed OCI CLI is available. Application pins are `oci==2.192.0`, `openai==3.55.1` and `python-dotenv==1.2.4`. If a pin is unavailable, resolve that with the source maintainer before proceeding.

To resume in a new terminal, enter the existing source checkout and run `source .venv/bin/activate`; do not recreate the environment. If you prefer Conda, use the alternative environment steps in the source README, then return at Task 3. Use one environment throughout the labs.

## Task 3: Create the OCI project

1. If you already have a workshop project, verify its settings and reuse it; skip the create command when they match. Otherwise, inspect the three JSON files under `config/`. They request 720-hour response/conversation retention, short-term compaction using `openai.gpt-oss-120b`, and enabled sandboxes with `MANAGED` networking and `DENY_ALL` outbound policy. Long-term memory is not enabled.
2. Confirm the compaction model is available in your chosen region. Select the matching OCI CLI profile and region in your terminal using your own credential setup; these must match the settings used by the SDK below.
3. Replace `YOUR_COMPARTMENT_OCID` and run the source project's create command. This creates a cloud resource.
```bash
<copy>
oci generative-ai generative-ai-project create \
  --compartment-id YOUR_COMPARTMENT_OCID \
  --display-name support-triage \
  --conversation-config file://config/project-conversation.json \
  --short-term-memory-optimization-config file://config/project-short-term-memory.json \
  --sandbox-config file://config/sandbox-project-deny-all.json
</copy>
```
4. Record the returned project OCID privately. Check the project in the OCI Console, including networking and retention. If the CLI does not recognize the options, check the installed CLI against the current Oracle documentation; the command was source-reviewed but not executed here. The Console is an alternative described in `labs/lab0.md`.

Expected result: a project with the required settings in the intended compartment and region. Review [Oracle's project creation documentation](https://docs.oracle.com/en-us/iaas/Content/generative-ai/create-project.htm) before creation; some settings have creation-time restrictions.

## Task 4: Configure secrets and settings

1. Create `.env` from the example only when `.env` does not already exist.
```bash
<copy>
test -e .env || cp .env.example .env
</copy>
```
2. Edit `.env` privately: set `OCI_GENAI_API_KEY` for the OCI model provider or `OPENAI_API_KEY` for OpenAI. Do not paste keys into Codex or terminal output.
3. Edit `config/settings.toml` yourself for this explicit setup task. The lab coding prompts must leave this file unchanged.

| Section | Required values |
| --- | --- |
| `[model]` | `provider` (`oci` or `openai`), `model_id` available to your account |
| `[oci]` | `region`, `project_id`, SDK `profile`, `auth` (`api_key` or `security_token`), and credential `config_file` if needed |
| `[sandbox]` | Retain the supplied runtime, shape and lifecycle defaults initially |

Expected result: `Settings.load()` can combine TOML configuration and the private environment file. OCI SDK credentials authenticate sandbox calls; the selected model API key authenticates Responses calls. Using OpenAI for the model still requires the OCI project and credentials.

## Task 5: Run preflight

1. Run quick preflight. It validates environment, configuration, credentials, project and permissions, and exercises model tool calling and conversation continuation. It creates no sandbox.
```bash
<copy>
python -m triage.preflight --quick
</copy>
```
2. After fixing any reported failures, run full preflight. This creates a test sandbox, checks runtime tools, transfers a 1 MiB file, tests blocked outbound access, and attempts cleanup.
```bash
<copy>
python -m triage.preflight
</copy>
```
3. Allow several minutes for startup and stopping. Follow every failure hint before continuing. If cleanup reports a sandbox still stopping, record its ID and delete it in the Console after it reaches `STOPPED`.

Expected result: all required checks pass. A slow cleanup warning does not block the offline build labs: record the sandbox ID and finish deletion in the cleanup lab. A failing environment, configuration, model or isolation check must be fixed before the corresponding live step.

**Done when:** you can identify the active interpreter, the selected provider and project, explain why `DENY_ALL` is used, and account for the preflight sandbox. Continue to [Lab 1](../workshops/freetier/index.html?lab=sandbox-tools).

## Acknowledgements

* **Author** - Luigi Saetta
* **Last Updated By/Date** - Luigi Saetta, November 2026
