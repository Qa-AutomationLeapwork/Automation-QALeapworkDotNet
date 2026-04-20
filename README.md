# Automation-QALeapworkDotNet

This repository contains a GitHub Actions workflow that triggers Leapwork schedules through the Leapwork Controller API, waits for execution to finish, and uploads a CSV summary as a workflow artifact.

## What This Repository Does

The workflow in [`.github/workflows/main.yml`](.github/workflows/main.yml):

- runs on pushes to `main`
- can also be started manually with `workflow_dispatch`
- calls the Leapwork Controller API to fetch available schedules
- starts each schedule with `runNow`
- polls run status and run items until execution is complete
- aggregates passed and failed results by flow title
- writes an `output.csv` file
- uploads `output.csv` as a GitHub Actions artifact

## Required GitHub Configuration

Before the workflow can run successfully, configure the following repository-level secret and variables in GitHub.

### Secret

Create this secret under:
`Settings` -> `Secrets and variables` -> `Actions` -> `Secrets`

| Name | Required | Description |
| --- | --- | --- |
| `ACCESS_KEY` | Yes | Leapwork API access key used in the `AccessKey` request header. |

### Variables

Create these variables under:
`Settings` -> `Secrets and variables` -> `Actions` -> `Variables`

| Name | Required | Example | Description |
| --- | --- | --- | --- |
| `BASE_URL` | Yes | `https://your-controller.company.com` | Base URL or host for the Leapwork Controller. |
| `TIME_DELAY` | Yes | `30` | Number of seconds to wait between polling calls while runs are still executing. |
| `ISDONESTATUSASSUCCESS` | Yes | `true` | Controls whether a Leapwork status of `Done` is treated as a pass or a failure. |

## BASE_URL Notes

The workflow accepts a few `BASE_URL` formats:

- full URL such as `https://controller.company.com`
- host plus path if your controller is hosted behind a path
- values without an explicit scheme, in which case the workflow assumes `https`

The workflow normalizes the URL before building API endpoints such as:

- `/api/v4/schedules`
- `/api/v4/schedules/{scheduleId}/runNow`
- `/api/v4/run/{runId}/status`
- `/api/v4/run/{runId}/runItemIds`
- `/api/v4/runItems/{runItemId}`

## How To Configure GitHub Actions

1. Open the repository on GitHub.
2. Go to `Settings` -> `Secrets and variables` -> `Actions`.
3. Add the `ACCESS_KEY` secret.
4. Add the `BASE_URL`, `TIME_DELAY`, and `ISDONESTATUSASSUCCESS` variables.
5. Commit and push changes to `main`, or run the workflow manually from the `Actions` tab.

## Example Configuration

Example repository settings:

| Type | Name | Example value |
| --- | --- | --- |
| Secret | `ACCESS_KEY` | `your-leapwork-access-key` |
| Variable | `BASE_URL` | `https://controller.company.com` |
| Variable | `TIME_DELAY` | `30` |
| Variable | `ISDONESTATUSASSUCCESS` | `true` |

Recommended starting values:

- use your Leapwork Controller HTTPS URL for `BASE_URL`
- start with `TIME_DELAY=30` unless you need faster polling
- set `ISDONESTATUSASSUCCESS=true` only if your team treats Leapwork `Done` status as a successful result

## How To Run The Workflow

### Automatic run

The workflow runs automatically when code is pushed to the `main` branch.

### Manual run

1. Open the `Actions` tab in GitHub.
2. Select the `Leapwork Action` workflow.
3. Click `Run workflow`.

## Output

When the workflow completes, it uploads an artifact named `output-csv`.

The generated CSV contains:

| Column | Description |
| --- | --- |
| `Flow Title` | Name of the Leapwork flow |
| `Passed Count` | Number of passed run items for that flow |
| `Failed Count` | Number of failed run items for that flow |

Example:

```csv
Flow Title,Passed Count,Failed Count
Login Flow,5,0
Checkout Flow,4,1
Regression Smoke,12,0
```

## Troubleshooting

### 401 or authorization errors

- verify that `ACCESS_KEY` is present and valid
- confirm the key has permission to access the Leapwork Controller API

### Invalid controller URL

- verify `BASE_URL` points to the correct Leapwork Controller
- include the correct hostname and any required path
- prefer `https://...` if you want to avoid relying on automatic scheme normalization

### Workflow hangs for too long

- reduce `TIME_DELAY` if polling is too slow
- confirm the target schedules are not waiting on unavailable environments or agents

### Results do not match expected pass/fail totals

- review the value of `ISDONESTATUSASSUCCESS`
- some Leapwork runs may return `Done` instead of `Passed`, and this flag changes how that status is counted

## Repository Structure

Current repository contents:

- [`.github/workflows/main.yml`](.github/workflows/main.yml): the GitHub Actions workflow
- [`README.md`](README.md): setup and usage guide
