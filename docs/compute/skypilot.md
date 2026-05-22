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

You then open a pull request to the `LEAP-batch-jobs` repository. Once the pull request is reviewed, a maintainer triggers the run from GitHub Actions.

The job runs on a GCP virtual machine, streams logs to the SkyPilot dashboard, and can post a Slack notification to `#leap-batch-jobs` when it finishes or fails.

GCP credentials are injected automatically, so you do not need to manage tokens manually.

## Write outputs

Batch job outputs should be written to either GCP or OSN, not local disk (lost when the VM shuts down).

### Writing to GCS

You can refer to [Data Tools](https://leap-stc.github.io/data/data_tools/#writing-to-gcs) for more information on writing to GCS. Use paths such as:

```text
gs://leap-persistent/<your_username>/
gs://leap-scratch/<your_username>/       # ephemeral, for intermediate data
```

### Writing to OSN

OSN uses an S3-compatible interface but requires configuring s3fs with the LEAP OSN endpoint. Add the following to your batch.py — OSN credentials are injected automatically by the workflow, so you can read them directly from environment variables:

```text
# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "s3fs",
#   "xarray",
#   "zarr",
# ]
# ///

import os
import s3fs
import xarray as xr

def main():
    # ... your computation ...

    fs = s3fs.S3FileSystem(
        key=os.environ["OSN_ACCESS_KEY_INBOX"],
        secret=os.environ["OSN_SECRET_KEY_INBOX"],
        client_kwargs={"endpoint_url": os.environ["OSN_ENDPOINT"]},
    )
    store = s3fs.S3Map("leap-pangeo-inbox/<your_username>/output.zarr", s3=fs)
    result.to_zarr(store, mode="w")

if __name__ == "__main__":
    main()
```

!!! note

    Use `s3fs.S3Map` to wrap the path when writing Zarr stores, and always use bytes mode `("wb"/"rb")` for direct file writes to avoid MissingContentLength errors.

## Pitfalls

### Do not commit secrets

- `batch.py` and `config.yml` are public. Never hardcode tokens, API keys, passwords, or other secrets in these files.

- Secrets committed to the repository may be visible in both GitHub and SkyPilot logs.

- If your job needs secrets, ask a maintainer to inject them at dispatch time using `job_env`.
