# LEAP data catalog

LEAP maintains a data catalog at [https://catalog.leap.columbia.edu](https://catalog.leap.columbia.edu).

The catalog includes data that's been ingested from public sources as well as datasets produced by LEAP. The catalog is publicly accessible.

## Data accessability

[LEAP data][where-data-lives] is either stored on a Google Cloud account (GCS) or on an Open Storage Network (OSN) pod. Data hosted on GCS is only available for access through the LEAP authenticated [JupyterHub][leap-jupyter-hub]. Data hosted on the OSN pod is publicly accessible and has no egress cost.

## Data Viewer

Zarr CF compliant datasets stored in the LEAP data catalog should be able to be viewed in the browser through the data viewer. This is an experimental feature that allows you to view Zarr data directly in the browser. An example of this can be seen [here](https://www.youtube.com/watch?v=OPFbZAsdKz0)
