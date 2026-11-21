# Need Help?

## Introduction

Use the failing check to locate a configuration, implementation or lifecycle problem.

Estimated Time: 5 minutes

### Objectives

* Recover from common setup and runtime failures.
* Distinguish local tests from live provider validation.

### Prerequisites

* Access to your source checkout, selected environment and the failing check's output.

## Task 1: Resolve the failing stage

| Symptom | Action |
| --- | --- |
| Wrong Python or missing package | Enter the source root, activate `.venv` or the Conda environment, then rerun dependency installation. |
| Missing model key | Check the selected provider and its corresponding key name in `.env`; do not print the key. |
| OCI 401 | Check the signing profile or refresh an expired session token; keep `oci.auth` aligned. |
| OCI 404 | Check project OCID, region and IAM permissions with your administrator. |
| OCI model endpoint error | Compare the source factory with the provider documentation as described in Lab 0; contact the maintainer before modifying provided code. |
| Model cannot call tools or continue responses | Check model capabilities, account access and stored response support; rerun quick preflight. |
| Sandbox quota or slow startup | Inspect existing resources and quota; adjust `sandbox.ready_timeout_seconds` if appropriate. |
| Outbound network is allowed | Do not proceed; use a project with `MANAGED/DENY_ALL`. |
| Missing lab module | Complete the corresponding Codex prompt before running downstream tests. |
| Evidence marked ✘ | Review the citation, file and first cited line; inspect the tools' error output and report parser. |
| Cleanup still STOPPING | Keep the sandbox ID; delete it in the OCI Console once `STOPPED`. |

Expected result: rerunning the affected acceptance command passes. Local tests use fakes and do not prove live provider access.

## Task 2: Resolve workshop rendering issues

Open `workshops/index.html` over HTTP, verify Oracle CDN access and retry with `?qa=true`. Use the navigation menu to switch labs. When reporting a problem, include the selected lab, URL and sanitized error; exclude credentials and customer bundle data.

## Acknowledgements

* **Author** - Luigi Saetta
* **Last Updated By/Date** - Luigi Saetta, November 2026
