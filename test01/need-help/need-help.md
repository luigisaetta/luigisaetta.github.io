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

## Acknowledgements

* **Author** - Luigi Saetta
* **Last Updated By/Date** - Luigi Saetta, November 2026
