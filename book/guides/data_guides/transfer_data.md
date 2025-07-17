(guides.data.ingestion)=

# Moving Data into Cloud Storage

This guide assumes a basic understanding of Data Ingestion and is intended to help intermediate to advanced users make intelligent long-term design decisions about they want to do with their data. If you are unfamiliar with the what, why, and how of ingestion, please first consult our [tutorial](tutorials.data.ingestion). But no matter what, if anything feels confusing or daunting your first instinct should always be reaching out to the [](support.data_compute_team)! This page is meant to serve as a high level reference, but troubleshooting will inevitably involve the specifics of your data.

Based on the 3 [types of data](explanation.data-policy.types) we host in the [](explanation.architecture.catalog) there are different ways of ingesting data:

- Linking an existing (public, egress-free) ARCO dataset to the [](explanation.architecture.catalog)
- Ingesting and transforming data into an ARCO copy on [](reference.infrastructure.buckets).
- (Work in Progress): Creating a [virtual zarr store](https://virtualizarr.readthedocs.io/en/latest/) from existing publicly hosted legacy format data (e.g. netcdf)

The end result should feel indistinguishable to the user (i.e. they just copy and paste a snippet and can immediately get to work). Given the wide variety of possible workflows, there are a couple of crucial design decisions that need to be made during the ingestion process:

1. *Migration strategy* - migration strategy depends on the load and accessibility of the data in its current form (how large is the dataset? Is it accessible online via HTTPs?). The Data and Compute team exists to help users find the best way to move their data.
1. *Where to migrate the data* - LEAP current supports ingestion both to LEAP owned Google Cloud Buckets and public OSN pods.

(guides.data.ingestion.workflow_selection)=

## Selecting a Workflow

Thinking about some of these design choices ahead of time greatly simplifies the back and forth needed by the DCT team when the request eventually comes in. Here are some high level guidelines and things to consider for your ingestion request.

### Choosing a final destination - OSN vs GCS

```{warning}
There might be special situations where it is beneficial/necessary to upload data to the [](reference.infrastructure.buckets) but we generally encourage data ingestion to the OSN Pod due to the public access and reduced running cost. See below for instructions.
```

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

(guide.data.upload_manual_deprecated)=

### Manual Upload - "pushing" vs "pulling"

If the source data is publicly available and accessable over https, you should create a [template feedstock](https://github.com/leap-stc/LEAP_template_feedstock) directly. If the data is located behind a firewall on an HPC center, the 'pull' based paradigm our feedstocks will not work. In this case we have an option to 'push' the data to a special "inbox" bucket (`'leap-pangeo-inbox'`) on the OSN Pod. This intermediate staging area makes the data accessible. From there an admin can move the data to another dedicated bucket and the data can be added to the catalog using the [template feedstock](https://github.com/leap-stc/LEAP_template_feedstock).

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

## Ingestion Instructions

Hopefully, the above guidelines were sufficient for selecting a workflow. Once a workflow is selected, you will need to actually be able to interface with Cloud Storage backends. There are many tools available to interact with cloud object storage. Here we for different scenarios.

### OSN Pod Ingestion

**Step by Step instructions**

- Reach out to the [](support.data_compute_team). They will contact the OSN pod admin and share bucket credentials for the `'leap-pangeo-inbox'` bucket.
- Authenticate to that bucket from a compute location that has access to your desired data and the internet. You can find instructions on how to authenticate [here](guides.data.external.authentication.config-files).
- Upload the data to the 'leap-pangeo-inbox' in **a dedicated folder** (note the exact name of that folder, it is important for the later steps). How you exactly achieve the upload will depend on your preference. Some common options include:
  - Open a bunch of netcdf files into xarray and use `.to_zarr(...)` to write the data to zarr.
  - Use fsspec or rclone to move an existing zarr store to the target bucket
    Either way the uploaded folder should contain one or more zarr stores!
- Once you have confirmed that all data is uploaded, ask an admin to move this data to the dedicated `'leap-pangeo-manual'` bucket on the OSN pod. They can do this by running [this github action](https://github.com/leap-stc/data-management/blob/main/.github/workflows/transfer.yaml), which requires the subfolder name from above as input.
- Once the data is moved, follow the instructions in the [template feedstock](https://github.com/leap-stc/LEAP_template_feedstock) to ["link an existing dataset"](https://github.com/leap-stc/LEAP_template_feedstock#linking-existing-arco-datasets) (The actual ingestion, i.e. conversion to zarr has been done manually in this case). Reach out to the [](support.data_compute_team) if you need support.

## Tools for Cloud Access

We currently have basic operations documented for three tools:

- GCloud SDK. One can directly interact with Google Cloud storage directly using the Google Cloud SDK and Command Line Interface. If you do not have this installed, please consult the [GCS Docs](https://cloud.google.com/sdk/docs/install).
- [fsspec](https://filesystem-spec.readthedocs.io/en/latest/) (and its submodules [gcsfs](https://gcsfs.readthedocs.io/en/latest/) and [s3fs](https://s3fs.readthedocs.io/en/latest/)) which provide filesystem-like access to local, remote, and embedded file systems from within a python session. Fsspec is also used by xarray under the hood.
  `fsspec` can be used to transfer small amounts of data to google cloud storage or OSN. If you have more than a few hundred MB's, it is worth using Rclone.
  `fsspec` should be installed by default on the **Base Pangeo Notebook** environment on the Jupyter-Hub. If you are working locally, you can install it with `pip or conda/mamba`. ex: `pip install fsspec`.

Using [`fsspec.open`](https://filesystem-spec.readthedocs.io/en/latest/api.html#fsspec.open) works similarly to the python builtin [`open`](https://docs.python.org/3/library/functions.html#open)

```python
with fsspec.open("gs://leap-scratch/funky-user/test.txt", mode="w") as f:
    f.write("hello world")
```

- [rclone](https://rclone.org/) which provides a Command Line Interface to many different storage backends.

```{admonition} Note on rclone documentation
---
class: tip, dropdown
---
Rclone is a very extensive and powerful tool, but with its many options it can be overwhelming (at least it was for Julius) at the beginning. We will only demonstrate essential options here, for more details see the [docs](https://rclone.org/). If however instructions here are not working for your specific use case, please reach out so we can improve the docs.
```

The below solutions fundamentally rely on the data being 'pushed' to the [](reference.infrastructure.buckets) which usually requires intervention on part of the [](explanation.data-policy.roles.data-expert). This stands in contrast to e.g. data ingestion where the [](explanation.data-policy.roles.data-expert) only has to work on the recipe creation and the data is 'pulled' in a reproducible way. For more information see [](explanation.data-policy).

(hub.data.list)=

### Reading and Writing Data

The first step for use of any tool is generally authentication. The JupyterHub is preconfigured to allow easy access to the LEAP Google Cloud buckets from within the notebooks, but you may need to authenticate certain tools or if you are working from outside the Hub. Check out the [authentication guide](guides.data.external.authentication) for detailed instructions on setting up these tools from wherever you are working.

Once authenticated, you can interface with cloud data.

`````{tab-set}
````{tab-item} Fsspec
The initial step in working with fsspec is to create a `filesystem` object which enables the abstraction on top of different object storage system.

```python
import fsspec

# for Google Storage 
fs = fsspec.filesystem('gs') # equivalent to gcsfs.GCSFileSystem()

# for s3
fs = fsspec.filesystem('s3') # equivalent to s3fs.S3FileSystem()
```

For **authenticated access** you need to pass additional arguments. In this case (for the m2lines OSN pod) we pass a custom endpoint and an [aws profile](guides.data.external.authentication):
```python
fs = fsspec.filesystem(
    's3',
    profile='<the_profile_name_you_picked>',  ## This is the profile name you configured above.
    client_kwargs={'endpoint_url': 'https://nyu1.osn.mghpcc.org '} # This is the endpoint for the m2lines osn pod
)
```

You can now use the `.ls` method to list contents of a bucket and prefixes.

You can e.g. list the contents of your personal folder on the persistent GCS bucket with

```python
fs.ls("leap-persistent/funky-user") # replace with your github username
```

````

````{tab-item} Rclone

To inspect a bucket you can use clone with the profile ('remote' in rclone terminology) set up [above](guides.data.external.authentication.config-files):

```shell
rclone ls <remote_name>:bucket-name/funky-user
```

````
`````

### Moving Data

`````{tab-set}
````{tab-item} fsspec/gcsfs

```python
import fsspec
fs = fsspec.filesystem('gs')
fs.put('<YOUR_DATA_FOLDER_ON_THE_JUPYTERHUB>', 'gs://leap-persistent/<YOUR_USERNAME>/<DATA_NAME>', recursive=True)
```
````

````{tab-item} Rclone

You can move directories from a local computer to cloud storage with rclone - make sure you are properly [authenticated](guides.data.external.authentication)!

```shell
rclone copy path/to/local/dir/ <remote_name>:<bucket-name>/funky-user/some-directory
```

You can also move data between cloud buckets using rclone

```shell
rclone copy \
 <remote_name_a>:<bucket-name>/funky-user/some-directory\
 <remote_name_b>:<bucket-name>/funky-user/some-directory
```
```{admonition} Copying single files
:class: note
To copy single files with rclone use the [copyto command](https://rclone.org/commands/rclone_copyto/) or copy the containing folder and use the `--include` or `--exclude` flags to select the file to copy.  
```

```{note}
Copying with rclone will stream the data from the source to your computer and back to the target, and thus transfer speed is likely limited by the internet connection of your local machine.
```

````
`````
