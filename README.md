# Automation-QALeapworkDotNet

This repository contains a GitHub Actions workflow that triggers Leapwork schedules through the Leapwork Controller API, waits for execution to finish, and uploads a CSV summary as a workflow artifact.

## What This Repository Does

The workflow in [`.github/workflows/main.yml`](.github/workflows/main.yml):

- can also be started manually with `workflow_dispatch`
- runs automatically every Wednesday at `8:00 PM` in the `Asia/Kolkata` timezone
- calls the Leapwork Controller API to fetch available schedules
- starts each schedule with `runNow`
- polls run status and run items until execution is complete
- writes a GitHub Actions job summary directly on the workflow run page
- writes a detailed `output.csv` file with schedule, flow, result, and agent details
- writes an aggregated `output-summary.csv` file with schedule counts and execution durations
- uploads both CSV files as a GitHub Actions artifact

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
5. Commit and push changes to `main`.
6. Run the workflow manually from the `Actions` tab or wait for the scheduled Wednesday `8:00 PM IST` run.

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

### Scheduled run

The workflow runs automatically every Wednesday at `8:00 PM` in the `Asia/Kolkata` timezone.

### Manual run

1. Open the `Actions` tab in GitHub.
2. Select the `Leapwork Action` workflow.
3. Click `Run workflow`.

## Output

When the workflow completes, it uploads an artifact named `output-csv`.

It also writes a GitHub Actions job summary to the run page that shows:

- triggered schedule count
- total flow count
- passed and failed flow totals
- overall execution duration
- a per-schedule summary table
- the generated artifact names

The artifact contains two CSV files.

### output.csv

This file contains one row per completed flow run.

| Column | Description |
| --- | --- |
| `Schedule Name` | Name of the Leapwork schedule that triggered the flow |
| `Flow Title` | Name of the Leapwork flow |
| `Run Item Status` | Raw Leapwork status returned for the run item |
| `Result` | Normalized result used by the workflow, such as `Passed` or `Failed` |
| `Executed Agent` | Agent or environment title used for the run, when provided by Leapwork |

Example:

```csv
Schedule Name,Flow Title,Run Item Status,Result,Executed Agent
Smoke Suite,Login Flow,Passed,Passed,Agent-01
Smoke Suite,Checkout Flow,Failed,Failed,Agent-02
Regression Suite,Search Flow,Done,Passed,Agent-03
```

### output-summary.csv

This file contains one overall summary row and one row for each triggered schedule.

| Column | Description |
| --- | --- |
| `Summary Type` | `Overall` for the full workflow or `Schedule` for an individual schedule |
| `Schedule Name` | Schedule name or `All Schedules` for the overall row |
| `Schedule ID` | Leapwork schedule ID for schedule rows |
| `Triggered Schedules Count` | Total number of triggered schedules, populated on the overall row |
| `Flow Count` | Number of completed flow results captured |
| `Passed Flow Count` | Number of flow rows classified as passed |
| `Failed Flow Count` | Number of flow rows classified as failed |
| `Schedule Duration Seconds` | Time spent processing an individual schedule |
| `Overall Duration Seconds` | End-to-end Leapwork action completion time |

Example:

```csv
Summary Type,Schedule Name,Schedule ID,Triggered Schedules Count,Flow Count,Passed Flow Count,Failed Flow Count,Schedule Duration Seconds,Overall Duration Seconds
Overall,All Schedules,,3,18,16,2,,254.42
Schedule,Smoke Suite,12345,,6,6,0,72.15,
Schedule,Regression Suite,67890,,8,6,2,121.84,
Schedule,Sanity Suite,24680,,4,4,0,60.43,
```

## Workflow Runtime

The artifact upload step uses `actions/upload-artifact@v7`, which aligns the workflow with GitHub's newer Node.js runtime support and avoids the deprecated Node.js 20 warning shown for older artifact action versions.

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
