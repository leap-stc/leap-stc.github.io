# Batch Jobs

LEAP provides a managed batch job system for running long or resource-intensive Python scripts on Google Cloud Platform (GCP), without needing to set up infrastructure yourself.

Batch jobs are defined in the [`leap-stc/LEAP-batch-jobs`](https://github.com/leap-stc/LEAP-batch-jobs) repository and run using SkyPilot and GitHub Actions. The `README.md` includes more in-depth instructions on how to run the batch job.

## When to use batch jobs

Use batch jobs when your workload is too heavy for JupyterHub, such as:

- Multi-hour or overnight compute jobs, including climatologies, model runs, or large aggregations
- Workloads that should run unattended and notify you when complete

If your script runs comfortably in a notebook in under approximately 15 minutes, you probably do not need to use the batch job system.

## How it works

To run a batch job, you write:

- A Python script, usually named `batch.py`
- A SkyPilot configuration file, usually named `config.yml`

You then open a pull request to the `LEAP-batch-jobs` repository. Once the pull request is ready, a maintainer triggers the run from GitHub Actions.

The job runs on a GCP virtual machine, streams logs to the SkyPilot dashboard, and can post a Slack notification to `#leap-batch-jobs` when it finishes or fails.

GCP credentials are injected automatically, so you do not need to manage tokens manually.

## Pitfalls

### Do not commit secrets

`batch.py` and `config.yml` are public. Never hardcode tokens, API keys, passwords, or other secrets in these files.

Secrets committed to the repository may be visible in both GitHub and SkyPilot logs.

If your job needs secrets, ask a maintainer to inject them at dispatch time using `job_env`.

### Write outputs to GCS

Batch job outputs should be written to Google Cloud Storage, not local disk.

Use paths such as:

````text
gs://leap-persistent/<your_username>/```
````
