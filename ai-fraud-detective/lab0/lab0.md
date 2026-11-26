# Lab 0: Setup and preflight

## Introduction

Prepare your machine and OCI project, select a model provider and verify the shared environment. All application infrastructure in this lab is provided.

Estimated Time: 5–7 minutes of hands-on work, plus reading, account preparation and provisioning waits.

### Objectives

* Install dependencies using Conda or venv.
* Configure an OCI project and either model provider.
* Pass quick and sandbox preflight checks.

### Prerequisites

* The accounts, tools, credentials, budget and permissions in the Introduction.
* An administrator has reviewed the [IAM guide](https://github.com/luigisaetta/ai-fraud-detective/blob/f6ee1a8b896f0369dc235e4cec1e94012c4b54eb/docs/oci-policies.md). Both paths need project and sandbox access; the OCI model path additionally needs API-key model permissions. Allow capacity for the investigation and preflight sandboxes.

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

This workshop was authored against source revision `f6ee1a8b896f0369dc235e4cec1e94012c4b54eb`. If your checkout differs, compare its guides and dependency versions before proceeding. Run every later command from this repository root, never from `labs/`.

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

If venv or ensurepip is unavailable, install your operating system's venv package for that Python version and retry. See the [source venv walkthrough](https://github.com/luigisaetta/ai-fraud-detective/blob/f6ee1a8b896f0369dc235e4cec1e94012c4b54eb/docs/setup-venv.md).

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

Save the returned project OCID. The network policy cannot be changed after creation; recreate an incorrectly configured project. Use a dedicated workshop project because cleanup considers every sandbox in it. These conversation settings govern the OCI model path; OpenAI stores its conversation separately. Keep the supplied project configuration for both paths because preflight checks it.

Expected result: an active project with these settings and sufficient sandbox capacity.

## Task 3: Configure the provider and credentials

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

For OpenAI, obtain the key through your OpenAI platform account. For OCI, obtain a Generative AI API key in the authorized compartment and confirm the model and condenser are available in the region. The [source IAM guide](https://github.com/luigisaetta/ai-fraud-detective/blob/f6ee1a8b896f0369dc235e4cec1e94012c4b54eb/docs/oci-policies.md) distinguishes user SDK permissions from API-key model permissions.

Both paths also require these settings:

* `[oci]`: set `region`, `project_id`, `config_file`, `profile` and `auth` to your project and SDK credentials. Use `auth = "api_key"` for signing keys or `auth = "security_token"` for an authenticated OCI session.
* `[sandbox]`: keep `runtime = "python-3.11"`, `shape = "SMALL"` and `lifetime_minutes = 60` initially. The runtime is independent of local Python 3.12. Leave the other supplied defaults unchanged.

Expected result: settings select exactly one investigation model provider while retaining OCI sandbox configuration. Both paths rejoin at Task 4.

## Task 4: Verify configuration and model tool calling

```bash
<copy>
python -m fraud_detective.preflight --quick
</copy>
```

This checks local dependencies, configuration, OCI credentials, project settings and permissions. It makes two model calls to verify a function call and continuation with `store=True` and `previous_response_id`; it creates no sandbox. Fix each failed check using its printed hint, then rerun.

Expected result: quick checks pass for your chosen provider. Model conversation storage follows that provider. The application does not explicitly configure OpenAI compaction; OCI project compaction applies only to OCI model conversations.

## Task 5: Verify sandbox execution and isolation

Run this yourself when ready to create a chargeable OCI test sandbox:

```bash
<copy>
python -m fraud_detective.preflight
</copy>
```

It checks runtime tools, transfers and reads back a 1 MiB file, verifies an outbound request is blocked, and requests cleanup. Provisioning may take several minutes; follow the progress messages. This is an acceptance command for your environment, not a claim that cloud checks were run during workshop authoring.

Expected result: `All checks passed`. A warning about cleanup still stopping is acceptable only if you record the sandbox ID and complete deletion later as described in Lab 3. Do not keep the test sandbox unless you need it. With `--keep` it expires after 20 minutes and still needs deletion after becoming `STOPPED`.

## Task 6: Read the coding rules

Read the source checkout's `AGENTS.md`. Codex must preserve provided infrastructure, existing tests, configuration and secrets; implement only the current lab modules. Settings come from `Settings.load()`, clients from the provided factories, and sandbox commands always have timeouts. Codex must not read `solutions/`.

You configure your own settings in this setup lab; later coding prompts must preserve them. Bundle reads and analysis run through `SandboxSession`. Tests use synthetic fixtures with fakes and never call OCI or a model.

Expected result: you can explain `DENY_ALL`, distinguish `.env` from `.venv`, and identify which service stores your investigation conversation. Continue to [Lab 1](../workshops/index.html?lab=lab1).

## Acknowledgements

* **Author** - L. Saetta
* **Last Updated By/Date** - L. Saetta, November 2026
