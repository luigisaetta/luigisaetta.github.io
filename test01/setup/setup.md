# Lab 0: Setup and Preflight

## Introduction

Prepare an isolated Python environment, configure an OCI Generative AI project and verify the model and sandbox. Commands below come from the source setup guides and provided preflight implementation. Live service commands remain unverified in this edition. **Review finding:** installing the source requirements during this review failed because the configured package index did not provide `openai==3.55.1`. Resolve that exact dependency with the source maintainer before attempting the executable labs; no substitute version has been validated. The review host also defaulted to Python 3.9, below the required 3.11, so use the supported interpreter specified below.

Estimated Time: 20 minutes

### Objectives

* Install the pinned application dependencies.
* Separate local secrets from versioned configuration.
* Distinguish quick preflight from full sandbox validation.

### Prerequisites

* Introduction completed.
* Python 3.12, Git, Codex and the OCI CLI installed.
* OCI SDK credentials configured privately; required tenancy access and API key ready.

## Task 1: Get the reviewed source

1. Clone into a new working directory; use a fresh clone for this workshop so existing work is preserved.
```bash
<copy>
git clone https://github.com/luigisaetta/support-bundle-triage-agent.git
cd support-bundle-triage-agent
git checkout --detach 228ad6756adcc41395ac76e86f0a2de48735dbf8
</copy>
```
2. Read `AGENTS.md` and `labs/lab0.md`. Keep the provided modules, tests and reference solutions unchanged. Ask Codex to work only on the current lab modules.

Expected result: a checkout matching the reviewed starting state, with four build modules absent. The detached checkout is suitable for local exercises; preserve your work before changing revisions.

## Task 2: Create the environment

1. Confirm the interpreter, create and activate the environment, then install dependencies. Run each line separately.
```bash
<copy>
python3.12 --version
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt -r requirements-dev.txt
oci --version
</copy>
```
2. Use `python` for the remaining lab commands. Activate this environment again in every new terminal.

Expected result: Python resolves inside `.venv`; package installation completes and the separately installed OCI CLI is available. Application pins are `oci==2.192.0`, `openai==3.55.1` and `python-dotenv==1.2.4`. If a pin is unavailable, resolve that with the source maintainer before proceeding.

## Task 3: Create the OCI project

1. Inspect the three JSON files under `config/`. They request 720-hour response/conversation retention, short-term compaction using `openai.gpt-oss-120b`, and enabled sandboxes with `MANAGED` networking and `DENY_ALL` outbound policy. Long-term memory is not enabled.
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

Expected result: all checks pass, with any cleanup warning resolved through the cleanup lab. These are participant acceptance criteria, not results observed during workshop authoring.

## Acknowledgements

* **Author** - Luigi Saetta
* **Last Updated By/Date** - Luigi Saetta, November 2026
