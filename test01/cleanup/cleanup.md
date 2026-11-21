# Clean Up Workshop Resources

## Introduction

Account for every sandbox, snapshot and project used by the workshop. Sandbox expiration stops execution but does not guarantee that the resource has been deleted.

Estimated Time: 5 minutes

### Objectives

* Delete workshop-owned cloud resources.
* Preserve shared resources and useful reports.
* Remove local secrets when the environment is retired.

### Prerequisites

* Access to the OCI Console and the resource IDs recorded during the workshop.
* Knowledge of which resources you created and which were pre-existing.

Keep a small private resource record while working:

| Resource | Where it came from | Cleanup responsibility |
| --- | --- | --- |
| Preflight sandbox | Full Lab 0 preflight | Automatic attempt; resolve any warning |
| New CLI sandbox | Lab 3 without `--sandbox-id` | Automatic attempt unless `--keep` |
| Attached sandbox | Lab 3 with `--sandbox-id` | Preserved by the CLI; owner cleans up |
| Snapshot | Optional `--snapshot` | Explicit removal when no longer needed |
| Project | Created or reused in Lab 0 | Delete only a dedicated workshop project |

The 5-minute estimate covers the cleanup actions; service state transitions may take longer. Cleanup is complete only after deletion is confirmed.

## Task 1: Resolve sandbox cleanup

1. Open your workshop project in the OCI Console in the same region used by the SDK.
2. Review sandboxes created by full preflight and each CLI run. Confirm automatically cleaned resources are gone.
3. For a sandbox you retained with `--keep`, attached to, or saw in a cleanup warning, stop it if running, wait for `STOPPED`, then delete it through the Console. Do this only for workshop-owned resources.
4. Check again after transitions complete. Keep track of any failure rather than assuming that expiry or a stop request completed deletion.

Expected result: no sandbox created for this workshop remains. The source lifecycle code stops before deletion and can report slow transitions. See [Oracle sandbox lifecycle guidance](https://docs.oracle.com/en-us/iaas/Content/generative-ai/create-sandbox.htm).

## Task 2: Remove retained cloud resources

1. If you used `--snapshot`, remove the resulting workshop snapshots after confirming you no longer need them.
2. Review retained conversations and responses under the provider account you selected. OCI project retention in this exercise is 720 hours; OpenAI provider storage is managed separately.
3. When the dedicated workshop project and its contents are no longer needed, delete it using the OCI Console after removing dependencies as directed by the service. Preserve projects shared with other work.
4. Check the relevant resource lists again and record any cleanup still pending.

Expected result: only resources you deliberately chose to retain remain. Region, permissions and resource dependencies can affect cleanup; use service error details to resolve failures.

## Task 3: Retire the local exercise

1. Preserve the reports and your implementation if needed, without including API keys or credential files.
2. Remove the exercise's local `.env` when you no longer need it. Revoke any API key issued solely for this workshop through its provider's controls. Preserve shared OCI credentials.
3. Deactivate the virtual environment. Remove the dedicated clone and environment only when your work is saved and they are no longer needed.
4. Review Git status before sharing any lab changes; exclude `.env`, reports containing sensitive data and private account settings.

Expected result: the exercise is closed with cloud cleanup checked and local secrets accounted for.

## Acknowledgements

* **Author** - Luigi Saetta
* **Last Updated By/Date** - Luigi Saetta, November 2026
