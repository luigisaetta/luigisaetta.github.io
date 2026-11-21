# Need Help?

## Introduction

Use this checklist to distinguish workshop loading problems, offline implementation failures and cloud access problems.

Estimated Time: 5 minutes

### Objectives

* Locate the failing stage.
* Collect a useful diagnostic without exposing secrets.

### Prerequisites

* The workshop URL and the lab or command that failed.

## Task 1: Check navigation and rendering

1. Serve the workshop over HTTP or HTTPS and open `workshops/freetier/index.html`.
2. Check network access to Oracle's CDN if the navigation, styling or copy controls do not appear. Reload and try another supported modern browser.
3. Report the lab title and browser error to the workshop author. For renderer QA, append `?qa=true` to the workshop URL.

Expected result: all seven navigation entries open readable English lessons. No account credentials are needed to browse the workshop.

## Task 2: Diagnose implementation and cloud failures

1. For an initial `ModuleNotFoundError` on one of the four lab modules, complete the corresponding build lab. If it was already built, confirm you are in the source root with the correct Python environment active.
2. For package installation failures, check the pinned versions and interpreter. For test failures, use the prompt and acceptance tests as the contract and change only your implementation.
3. For preflight failures, follow the printed hint. Check matching project ID, region and SDK profile for 404 errors; verify permissions with your administrator for authorization errors.
4. For network isolation failures, check that the project uses the expected `MANAGED/DENY_ALL` policy. For sandbox startup timeouts, inspect the lifecycle state and the source setting `ready_timeout_seconds` before adjusting it.
5. For cleanup warnings, follow the cleanup lab. Record the operation, sanitized error and resource state; never include keys, private credential files or real customer bundles in a help request.

Expected result: a specific failing stage and a safe diagnostic. Source guides in `labs/`, prompts in `prompts/` and tests in `tests/` provide the exercise contracts. This workshop does not register an Oracle catalog support channel.

### Quick recovery checklist

| Symptom | Next action |
| --- | --- |
| `python` points outside `.venv` | Enter the source root, activate `.venv`, then check `sys.executable` |
| A pinned package cannot be installed | Check Python, package index and the exact requirements file; ask the source maintainer before changing pins |
| An expected initial import failure persists | Build the matching module, or confirm the working directory and active environment if it already exists |
| Report JSON is rejected | Check provider support for strict structured output and the returned response; keep the schema and existing tests intact |
| Report contains a cross beside evidence | Inspect citation parsing, file existence and the first cited line; do not treat the mark alone as a diagnosis |
| CLI reports success with a cleanup warning | Preserve the reports; record the sandbox ID and complete deletion once `STOPPED` |

For timing, use the 5-minute estimates in this LiveLab. The reviewed source README and lab overview still contain the older two-hour plan.

## Acknowledgements

* **Author** - Luigi Saetta
* **Last Updated By/Date** - Luigi Saetta, November 2026
