# Setup and Preflight

## Introduction

Prepare your source checkout, Python environment, OCI project and model configuration. You build no application module in this lab. Run commands one at a time from the source repository root, with your chosen environment active.

Estimated Time: 5 minutes

Installation and sandbox lifecycle waits are additional elapsed time.

### Objectives

* Prepare either venv or Conda and install the same dependencies.
* Configure either OCI Generative AI or OpenAI while retaining OCI Sandbox execution.
* Pass quick and full preflight checks.

### Prerequisites

* The accounts, permissions, CLI, Git and Codex described in the Introduction.
* For venv: an installed Python 3.11+ interpreter (examples use `python3.12`). For Conda: Conda installed; no separate Python installation is required.
* Install the OCI CLI using the [official CLI quickstart](https://docs.oracle.com/en-us/iaas/Content/API/SDKDocs/cliinstall.htm), and configure an [OCI SDK/CLI profile](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/sdkconfig.htm).

## Task 1: Open the source checkout

1. If you do not already have the project, clone it:

```bash
<copy>
git clone https://github.com/luigisaetta/support-bundle-triage-agent.git
</copy>
```

2. Open the repository root. For an existing checkout, use its location instead:

```bash
<copy>
cd support-bundle-triage-agent
</copy>
```

Read `AGENTS.md`. Do not change provided modules, existing tests or the rules file. The reviewed revision has the provided infrastructure and tests; the lab modules must be built. Some source overview text says implementations are included: inspect your checkout rather than deleting files to force an expected initial failure.

## Task 2: Choose venv or Conda

### Option A: Python venv

1. Confirm your installed interpreter:

```bash
<copy>
python3.12 --version
</copy>
```

2. Create the environment once. Substitute your supported interpreter name if needed:

```bash
<copy>
python3.12 -m venv .venv
</copy>
```

3. Activate it:

```bash
<copy>
source .venv/bin/activate
</copy>
```

If `venv` or `ensurepip` is unavailable, install your operating system's venv package for that Python version and retry. The `.venv/` directory contains Python packages; `.env` is a separate secrets file.

### Option B: Conda

1. Create an environment with Python 3.12:

```bash
<copy>
conda create -n support-bundle-triage-agent python=3.12 -y
</copy>
```

2. Activate it:

```bash
<copy>
conda activate support-bundle-triage-agent
</copy>
```

### Shared path: install and verify

Both options rejoin here:

```bash
<copy>
python -c "import sys; print(sys.executable)"
python -m pip install -r requirements.txt -r requirements-dev.txt
oci --version
</copy>
```

The interpreter path must belong to the selected environment and dependency installation must finish successfully. The requirements file pins runtime dependencies; use it rather than installing only the looser minimum versions in `pyproject.toml`. The OCI CLI is a separate prerequisite.

In a new terminal, enter this checkout and repeat your activation command; do not recreate the environment. Leave venv with `deactivate`, or Conda with `conda deactivate` when finished. Deactivation does not stop cloud resources.

## Task 3: Prepare the OCI project for either model route

1. Ask your administrator for permission to create/read Generative AI projects and create/list/manage sandboxes in your compartment. Use a region with the required services and model access. Configure `~/.oci/config` using your own credentials; never paste signing keys into the workshop.
2. Create a project using the versioned JSON files. Replace `YOUR_COMPARTMENT_OCID`. Ensure the CLI is using your intended region and profile before running:

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

Alternatively create the project in the OCI Console with these same settings, or verify an existing project before reuse:

| Setting | Required workshop value |
| --- | --- |
| Conversations/responses | 720-hour retention in `project-conversation.json` |
| Short-term memory | Enabled; condenser `openai.gpt-oss-120b` in `project-short-term-memory.json` |
| Long-term memory | Not enabled |
| Sandboxes | Enabled, network `MANAGED`, policy `DENY_ALL` |

Save the returned project OCID. If the network policy is wrong, create a new project: the source guide states that it cannot be changed afterward. Outbound sandbox traffic is denied; your local client still connects to OCI APIs. The bundled preflight checks the OCI project's conversation/memory settings even with an OpenAI model, so use this project configuration on both routes. Those settings do not configure OpenAI's conversation storage.

## Task 4: Configure your model provider

1. Create the secrets file **only if it does not already exist**:

```bash
<copy>
cp -n .env.example .env
</copy>
```

2. Choose one branch. Edit the existing `[model]` section in `config/settings.toml`; do not append duplicate sections. Replace the uppercase placeholder with a model available to your account that supports Responses, function calling, stored response chaining and strict structured output.

| Choice | Model setting | Secret in `.env` | Client behavior |
| --- | --- | --- | --- |
| OCI Generative AI | `provider = "oci"`; OCI model ID | `OCI_GENAI_API_KEY` | Uses region-derived endpoint and project OCID |
| OpenAI | `provider = "openai"`; OpenAI model name | `OPENAI_API_KEY` | Uses the OpenAI endpoint; OCI SDK still handles sandboxes |

### Option A: OCI Generative AI

Obtain the model key following [OCI Generative AI API keys](https://docs.oracle.com/en-us/iaas/Content/generative-ai/api-keys.htm). This is different from the OCI IAM signing key used by the SDK. Put its value in `OCI_GENAI_API_KEY` in `.env`.

```toml
[model]
provider = "oci"
model_id = "YOUR_OCI_MODEL_ID"
```

The provided `triage/llm.py` constructs `https://inference.generativeai.{region}.oci.oraclecloud.com/openai/v1` and passes `project=oci_project_id`. The currently published API-key documentation shows a different SDK base path, `/20231130/actions/v1`. This workshop preserves the source factory; compatibility of its endpoint and your selected model must be established by quick preflight. If it fails, have the project maintainer resolve the discrepancy before continuing on this route; do not silently edit the provided module.

### Option B: OpenAI

Create a key in your [OpenAI API account](https://platform.openai.com/api-keys), ensure model access and billing are configured, and put its value in `OPENAI_API_KEY` in `.env`.

```toml
[model]
provider = "openai"
model_id = "YOUR_OPENAI_MODEL_NAME"
```

The client uses OpenAI for the model and stored responses. OCI project retention/compaction does not apply, and this application does not request OpenAI compaction.

### Shared configuration and acceptance

For **both** branches edit `[oci]`: set `region`, your actual `project_id`, `config_file`, `profile`, and `auth` (`api_key` or `security_token`, matching your SDK profile). Do not reuse identifiers already present in an example checkout. For token authentication, maintain a valid OCI session and use the same profile when provisioning. Leave `[sandbox]` defaults initially; `ready_timeout_seconds` controls how long startup may wait. API keys remain only in `.env`; never commit them.

Both provider branches rejoin at the same acceptance command in Task 5. Switching providers changes the model ID and model secret, not the OCI sandbox credentials or execution infrastructure.

## Task 5: Run preflight

1. Check the environment, configuration, OCI credentials, project, permissions and model:

```bash
<copy>
python -m triage.preflight --quick
</copy>
```

Quick mode makes two model requests to check tool calling and `previous_response_id`; it creates no sandbox. Fix each reported failure and rerun. It is not an offline check and model usage may incur charges.

2. Run the full check when ready to create a test sandbox:

```bash
<copy>
python -m triage.preflight
</copy>
```

It starts a sandbox, checks runtime tools, transfers a 1 MiB file, checks outbound blocking and attempts cleanup. Startup/stop can take several minutes. Continue only when all checks pass, allowing a slow-cleanup warning after following its instructions to delete the sandbox from the OCI Console once `STOPPED`. Do not ignore network-isolation failures.

Expected result: a working model conversation and an isolated test sandbox, with cleanup completed or explicitly tracked. Continue to Lab 1. These live checks are participant acceptance steps; neither provider nor sandbox execution was live-tested while authoring this workshop.

## Acknowledgements

* **Author** - Luigi Saetta
* **Last Updated By/Date** - Luigi Saetta, November 2026
