# Need Help?

## Introduction

Use the failing check and its hint to isolate a problem before retrying a live investigation. Work from the participant repository root with the chosen Python environment active. Keep credentials and case reports private when asking for assistance.

Estimated Time: 5–10 minutes as needed.

### Objectives

* Recover environment, configuration, provider and sandbox failures.
* Keep earlier lab contracts and source rules intact.
* Complete cleanup after a failed or interrupted run.

### Prerequisites

* The lab number, failing command and redacted error message.
* Access to your participant checkout and its `AGENTS.md`.

## Task 1: Diagnose setup or service access

| Symptom | Action |
| --- | --- |
| Wrong Python or missing packages | Reactivate `conda activate ai-fraud-detective` or `source .venv/bin/activate`; reinstall both requirements files with `python -m pip`. |
| Configuration check fails | Fix the named setting in `config/settings.toml` or the secret privately in `.env`. Exported environment variables override `.env`. |
| Project returns 404 / NotAuthorizedOrNotFound | Check the project ID, region, OCI profile/session and compartment permissions with your administrator. |
| OpenAI model request returns 401/403/404 | Check `OPENAI_API_KEY`, model access and `[model] provider/model_id`. OCI credentials do not grant OpenAI access. |
| OCI model request returns 401/403/404 | Check `OCI_GENAI_API_KEY`, region/model availability and API-key principal policy. SDK user credentials are separate. |
| Sandbox can be listed but not created/deleted | The administrator must check `manage generative-ai-sandbox`; `use` alone does not grant lifecycle operations. |
| Sandbox creation returns 429 | Check quota/capacity and identify your own leftover sandboxes. |
| Network isolation fails | Verify `MANAGED` / `DENY_ALL`. Create a correctly configured project if necessary; its network policy is immutable. |
| Sandbox start times out | Check service state/capacity; review `sandbox.ready_timeout_seconds` before retrying. |
| Lab 4 API or model unavailable | Verify the independent `[decisions]` configuration and provider access. Skip this optional lab if unavailable. |

After correcting the issue, rerun:

```bash
<copy>
python -m fraud_detective.preflight --quick
</copy>
```

Only run full preflight when ready to create a real test sandbox. Review the [source administrator guide](https://github.com/luigisaetta/ai-fraud-detective/blob/f6ee1a8b896f0369dc235e4cec1e94012c4b54eb/docs/oci-policies.md) for identity, policies, regions and capacity.

Expected result: the relevant preflight checks pass; failures have a specific next action.

## Task 2: Recover a coding lab

Run the current lab's documented acceptance command. A missing new module is expected before implementation, but an earlier lab's failure must be fixed before continuing. Ask Codex to inspect the relevant prompt, public interfaces, provided classes and existing acceptance tests. Fix your lab module; do not change or skip existing tests, modify provided infrastructure, or have Codex read `solutions/`.

If you need to catch up, ask the instructor for assistance. The source documents a human-operated reference-solution path, but this authoring workflow has not read, imported or copied those solutions. This workshop keeps the coding path and its review steps; no solution-copy command is required.

Expected result: the current and prerequisite lab tests pass, and you can explain the implementation. Keep formatting and lint checks from the lab as part of completion.

## Task 3: Recover an investigation and finish cleanup

A missing `.sha256` reference or changed archive must stop before sandbox creation. Restore the intended bundle and its trusted reference rather than bypassing integrity checks. Do not replace a trusted reference merely to make a modified archive pass.

If the report has failed citation marks, inspect the cited file/line through the sandbox tools and review the trace. A successful existence check still requires human interpretation. Investigate unsupported conclusions instead of treating a schema-valid report as proof.

List resources before deleting anything:

```bash
<copy>
python -m fraud_detective.cleanup
</copy>
```

After reviewing the list for your dedicated project:

```bash
<copy>
python -m fraud_detective.cleanup --delete
</copy>
```

Confirm only resources you own. Rerun once stopping sandboxes become stopped. An attached sandbox is preserved by the CLI; an expired sandbox is stopped rather than deleted. Snapshots and dedicated projects need separate lifecycle review in the OCI Console. See Task 7 in [Lab 3](../workshops/index.html?lab=lab3).

Expected result: owned sandboxes are deleted and any remaining resources have an explicit owner and cleanup action. Resume the relevant lab after resolving its acceptance checks.

## Acknowledgements

* **Author** - L. Saetta
* **Last Updated By/Date** - L. Saetta, November 2026
