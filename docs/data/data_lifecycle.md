# Data Lifecycle

This guide assumes a basic understanding of Data Ingestion and is intended to help intermediate to advanced users make intelligent long-term design decisions about they want to do with their data. If you are unfamiliar with the ***what, why, and how*** of ingestion, please first consult our [tutorial](tutorials.data.ingestion). But no matter what, if anything feels confusing or daunting your first instinct should always be reaching out to the [](support.data_compute_team)! This page is meant to serve as a high level reference, but troubleshooting will inevitably involve the specifics of your data.

Based on the 3 [types of data](explanation.data-policy.types) we host in the [](explanation.architecture.catalog) there are different ways of ingesting data:

- Linking an existing (public, egress-free) ARCO dataset to the [](explanation.architecture.catalog)
- Ingesting and transforming data into an ARCO copy on [](reference.infrastructure.buckets).
- (Work in Progress): Creating a [virtual zarr store](https://virtualizarr.readthedocs.io/en/latest/) from existing publicly hosted legacy format data (e.g. netcdf)

```{note}
You want to have a specific dataset to explore or analyze? There is a chance that somebody else at LEAP has already worked with the data! So the first thing to look for data should always be a visit to the [LEAP Data Catalog](explanation.architecture.catalog).
```

The end result should feel indistinguishable to the user (i.e. they just copy and paste a snippet and can immediately get to work). Given the wide variety of possible workflows, there are a couple of crucial design decisions that need to be made during the ingestion process:

1. *Migration strategy* - migration strategy depends on the load and accessibility of the data in its current form (how large is the dataset? Is it accessible online via HTTPs?). The Data and Compute team exists to help users find the best way to move their data.
1. *Where to migrate the data* - LEAP current supports ingestion both to LEAP owned Google Cloud Buckets and public OSN pods.

## Selecting a Workflow

Thinking about some of these design choices ahead of time greatly simplifies the back and forth needed by the DCT team when the request eventually comes in. Here are some high level guidelines and things to consider for your ingestion request.

### Choosing a final destination - OSN vs GCS

```{warning}
There might be special situations where it is beneficial/necessary to upload data to the [](reference.infrastructure.buckets) but we generally encourage data ingestion to the OSN Pod due to the public access and reduced running cost. See below for instructions.
```

LEAP owns two Google Cloud buckets (`leap-persistent` and `leap-scratch`) and also has an allocation of storage on an OSN pod. We have complete access control to GCS. OSN allows s3-like cloud storage that has no egress fees, which means that you can share data with the public or outside colaborators without any cost per request.
Details on both OSN and GCS cloud storage buckets can be found in the [infrastructure guide](reference.infrastructure.catalog).

For some general guidelines on choosing:

GCS:
\- You want to move data from your Jupyter-Hub home directory to the cloud.
\- You don't need the data to be accessed outside of the Jupyter-Hub.
\- This data is a work-in-progress and might be regenerated or modified.

OSN:
\- You want your data to be publicly accessible outside of the Jupyter-Hub.
\- You need to move data from your Jupyter-Hub home directory to more persistent storage.
\- You data does not fit into the Zarr model.

If you believe OSN better fits your data use case, please contact the data-and-compute team on slack. Then follow the instructions on [OSN Ingestion](guides.data.osn_pod).

### Manual Upload - "pushing" vs "pulling"

If the source data is publicly available and accessible over https, you should create a [template feedstock](https://github.com/leap-stc/LEAP_template_feedstock) directly. If the data is located behind a firewall on an HPC center, the 'pull' based paradigm our feedstocks will not work. In this case we have an option to 'push' the data to a special "inbox" bucket (`'leap-pangeo-inbox'`) on the OSN Pod. This intermediate staging area makes the data accessible. From there an admin can move the data to another dedicated bucket and the data can be added to the catalog using the [template feedstock](https://github.com/leap-stc/LEAP_template_feedstock).

We discourage manually moving datasets to our cloud storage as much as possible since it is hard to reproduce these datasets at a future point (if e.g. the dataset maintainer has moved on to a different position) (see [](explanation.data-policy.reproducibility). We will always [prioritize unblocking your work](explanation.code-policy.dont-let-perfect-be-the-enemy-of-good).

<!-- Fundamentally the 'pushing' of datasets relies on two components:

- Setting up permissions so that you can read/write to the [](reference.infrastructure.buckets) - Several methods to get permissions are described in [](guides.data.external.authentication).
- Initiating a data transfer from your 'local' machine (e.g. your laptop, a server, or HPC Cluster). You can check on some methods below. -->

### Transfer scenarios

small: KB's to 100's of MB
medium: 1 to \<100GB

#### Move data from Jupyter-Hub to GCS

- Data volume:
  - small: `fsspec/gcsfs`
  - medium: `rclone`

#### Move data from Jupyter-Hub to OSN

- If the data volume is:
  - small: `rclone`
  - medium: `rclone`

#### Move data from Laptop/HPC to OSN

- If the data volume is:
  - small: `rclone`
  - medium: `rclone`
  - large: `rclone`

## Available Tooling

Depending on the above guidelines, you will need access to different tooling. **TODO** *Expand upon available tools and how to set them up, somehow sync with [](guide.data.working)*

### OSN Pod Ingestion

**Step by Step instructions**

- Reach out to the [](support.data_compute_team). They will contact the OSN pod admin and share bucket credentials for the `'leap-pangeo-inbox'` bucket.
- Authenticate to that bucket from a compute location that has access to your desired data and the internet. You can find instructions on how to authenticate [here](guides.data.external.authentication.config-files).
- Upload the data to the 'leap-pangeo-inbox' in **a dedicated folder** (note the exact name of that folder, it is important for the later steps). How you exactly achieve the upload will depend on your preference. Some common options include:
  - Open a bunch of netcdf files into xarray and use `.to_zarr(...)` to write the data to zarr.
  - Use fsspec or rclone to move an existing zarr store to the target bucket
    Either way the uploaded folder should contain one or more zarr stores!
    A typical workflow for the above steps might look like:

```python
import xarray as xr
import fsspec
import zarr

ds = xr.tutorial.open_dataset("air_temperature", chunks={})
ds_processed = ds.mean(...).resample(...)  # sample modifications to data

# define our credentials, bucket name and dataset path
osn_bucket_name = "leap-pangeo-inbox"
osn_key = "<ask DCT team>"
osn_secret = "<ask DCT team>"
endpoint_url = "https://nyu1.osn.mghpcc.org"
dataset_path = f"{osn_bucket_name}/{dataset_name}"

# Here we are using fsspec/s3fs to pass our OSN credentials to Zarr.
# Note: If you get an error like: TypeError: Unsupported type for store_like: 'FSMap'`. It is because zarr-python does not currently support the older fsspec FSMap object style. https://github.com/zarr-developers/zarr-python/issues/2706

fs = fsspec.filesystem(
    "s3",
    key=osn_key,
    secret=osn_secret,
    client_kwargs={"endpoint_url": endpoint_url},
    asynchronous=True,
)
store = zarr.storage.FsspecStore(fs, path=dataset_path)

zarr_format = 3

ds_processed.to_zarr(store=store, zarr_format=zarr_format, consolidated=False)

# Note: Your data can be read anywhere, by anyone!
# roundtrip = xr.open_zarr(f"{dataset_path}', consolidated=False)
```

- Once you have confirmed that all data is uploaded, ask an admin to move this data to the dedicated `'leap-pangeo-manual'` bucket on the OSN pod. They can do this by running [this github action](https://github.com/leap-stc/data-management/blob/main/.github/workflows/transfer.yaml), which requires the subfolder name from above as input.
- Once the data is moved, follow the instructions in the [template feedstock](https://github.com/leap-stc/LEAP_template_feedstock) to ["link an existing dataset"](https://github.com/leap-stc/LEAP_template_feedstock#linking-existing-arco-datasets) (The actual ingestion, i.e. conversion to zarr has been done manually in this case). Reach out to the [](support.data_compute_team) if you need support.
