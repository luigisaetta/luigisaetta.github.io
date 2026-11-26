# Lab 0: Setup and preflight

## Introduction

An AI investigator needs both reasoning and a place to execute analysis. You will prepare Python with Conda or venv, use the OCI CLI and OCI SDK credentials to access an OCI Generative AI project and its OCI Sandboxes, and connect the OpenAI Responses API through OpenAI or the alternative OCI Generative AI provider. Preflight checks verify model tool calling and sandbox isolation before you build the agent with Codex.

Estimated Time: 5–7 minutes of hands-on work, plus reading, account preparation and provisioning waits.

### Objectives

* Install dependencies using Conda or venv.
* Configure an OCI project and either model provider.
* Pass quick and sandbox preflight checks.

### Prerequisites

* The accounts, tools, credentials, budget and permissions in the Introduction.
* An administrator has reviewed the [IAM guide](https://github.com/luigisaetta/ai-fraud-detective/blob/main/docs/oci-policies.md). Both paths need project and sandbox access; the OCI model path additionally needs API-key model permissions. Allow capacity for the investigation and preflight sandboxes.

## Task 1: Get the source and activate Python

Skip cloning if you already have the participant repository. Otherwise run these commands one at a time:

```bash
<copy>
git clone https://github.com/luigisaetta/ai-fraud-detective.git
</copy>
```

```bash
<copy>
cd ai-fraud-detective
</copy>
```

Run every later command from this repository root, never from `labs/`. The walkthrough uses Python 3.12; the package requires Python 3.11 or later.

Choose one environment path.

**Conda:** no separately installed Python is needed. Create the environment once:

```bash
<copy>
conda create -n ai-fraud-detective python=3.12 -y
</copy>
```

Activate it now and whenever you return in a new terminal:

```bash
<copy>
conda activate ai-fraud-detective
</copy>
```

**venv:** start outside any other virtual environment. Confirm that your installed interpreter reports Python 3.12.x, then create the environment once:

```bash
<copy>
python3.12 --version
python3.12 -m venv .venv
</copy>
```

Activate it now and whenever you return to the repository in a new terminal:

```bash
<copy>
source .venv/bin/activate
</copy>
```

If venv or ensurepip is unavailable, install your operating system's venv package for that Python version and retry. See the [source venv walkthrough](https://github.com/luigisaetta/ai-fraud-detective/blob/main/docs/setup-venv.md).

Both paths rejoin here. Confirm the active interpreter and install the pinned application dependencies and development tools:

```bash
<copy>
python -c "import sys; print(sys.executable)"
python -m pip install -r requirements.txt -r requirements-dev.txt
oci --version
</copy>
```

Expected result: Python points into your chosen environment, installation succeeds and the OCI CLI is available. To resume later, enter this checkout and repeat only its activation command. To leave the environment use `conda deactivate` or `deactivate`, respectively.

## Task 2: Create the Generative AI project

OCI is required even with OpenAI as the model provider. Configure the OCI CLI/SDK credentials in `~/.oci/config` using your organization's approved process. Use a region supporting sandboxes and matching your project. The source examples use `eu-frankfurt-1`; availability and quota must be checked for your tenancy.

Replace `YOUR_COMPARTMENT_OCID` before executing:

```bash
<copy>
oci generative-ai generative-ai-project create \
  --compartment-id YOUR_COMPARTMENT_OCID \
  --display-name fraud-detective \
  --conversation-config file://config/project-conversation.json \
  --short-term-memory-optimization-config file://config/project-short-term-memory.json \
  --sandbox-config file://config/sandbox-project-deny-all.json
</copy>
```

Alternatively create the project in the OCI Console with these same settings. If using an existing project, verify them before reuse:

| Setting | Required value from supplied configuration |
| --- | --- |
| Conversations and responses | Retained 720 hours |
| Short-term memory | Enabled, condenser `openai.gpt-oss-120b` |
| Long-term memory | Not enabled |
| Sandboxes | Enabled, `MANAGED` networking, `DENY_ALL` outbound policy |

Save the returned project OCID. The network policy cannot be changed after creation; recreate an incorrectly configured project. Use a dedicated workshop project because cleanup considers every sandbox in it. Keep the supplied project configuration for both provider paths because preflight checks it.

Expected result: an active project with these settings and sufficient sandbox capacity.

## Task 3: Configure the provider and credentials

Choose where the model runs. OCI supplies the execution workspace either way; the selected model provider stores the investigation conversation.

| Layer | OpenAI, usual path | OCI Generative AI, alternative |
| --- | --- | --- |
| Investigation model | OpenAI | OCI Generative AI |
| Stored conversation | OpenAI platform/account settings | OCI project settings from Task 2 |
| Conversation compaction | Not explicitly configured by this application | OCI project short-term memory |
| Analysis execution | OCI Sandbox | OCI Sandbox |

Neither path enables long-term memory across investigations. Blocking sandbox outbound traffic contains the analysis code; your local application still sends selected tool output to the model provider and saves reports locally. Use the invented workshop data.

Create a secrets file only if `.env` does not already exist:

```bash
<copy>
cp .env.example .env
</copy>
```

Edit it privately. Never paste keys into chat or commit them. Environment variables already exported in your shell take precedence over `.env`. Edit the existing sections of `config/settings.toml`; do not append duplicate tables.

| Choice | Secret in `.env` | `[model]` settings |
| --- | --- | --- |
| OpenAI, usual path | `OPENAI_API_KEY` | `provider = "openai"`, `model_id = "gpt-6.1-sol"` |
| OCI Generative AI, alternative | `OCI_GENAI_API_KEY` | `provider = "oci"`, `model_id = "openai.gpt-6.1-sol"` |

The checked-in settings default to OCI: explicitly select OpenAI for the usual path. These model IDs are the source examples; your key must have access. Use a supported tool-calling Responses model if an example is unavailable, then verify it in Task 4.

For OpenAI, obtain the key through your OpenAI platform account. For OCI, obtain a Generative AI API key in the authorized compartment and confirm the model and condenser are available in the region. The [source IAM guide](https://github.com/luigisaetta/ai-fraud-detective/blob/main/docs/oci-policies.md) distinguishes user SDK permissions from API-key model permissions.

Both paths also require these settings:

* `[oci]`: set `region`, `project_id`, `config_file`, `profile` and `auth` to your project and SDK credentials. Use `auth = "api_key"` for signing keys or `auth = "security_token"` for an authenticated OCI session.
* `[sandbox]`: leave the supplied defaults unchanged. The sandbox's Python runtime is independent of your local environment.

Expected result: settings select exactly one investigation model provider while retaining OCI sandbox configuration. Both paths rejoin at Task 4.

## Task 4: Verify configuration and model tool calling

```bash
<copy>
python -m fraud_detective.preflight --quick
</copy>
```

This checks local dependencies, configuration, OCI credentials, project settings and permissions. It also calls the selected model to check that it can request a tool and continue the conversation; it creates no sandbox. Fix each failed check using its printed hint, then rerun.

Expected result: quick checks pass for your chosen provider.

## Task 5: Verify sandbox execution and isolation

Run this yourself when ready to create a chargeable OCI test sandbox:

```bash
<copy>
python -m fraud_detective.preflight
</copy>
```

It checks runtime tools, transfers and reads back a file, verifies an outbound request is blocked, and requests cleanup. Provisioning may take several minutes; follow the progress messages.

Expected result: `All checks passed`. If cleanup reports that the sandbox is still stopping, record its ID and complete deletion in Lab 3.

## Task 6: Read the coding rules

Read the source checkout's `AGENTS.md`. Codex must preserve provided infrastructure, existing tests, configuration and secrets, and implement only the current lab modules. Do not have Codex read `solutions/` or instructor-only case documentation.

You configure your own settings in this setup lab; later coding prompts must preserve them. Application tests use synthetic fixtures and fake services so you can check code without cloud calls.

Expected result: you can explain `DENY_ALL`, distinguish `.env` from `.venv`, and identify which service stores your investigation conversation. Continue to [Lab 1](../workshops/index.html?lab=lab1).

## Acknowledgements

* **Author** - L. Saetta
* **Last Updated By/Date** - L. Saetta, November 2026
