(guide.data.transfer)=

# Transferring Data into Cloud Storage

We distinguish between two primary *types* of data to upload: "Original" and "Published" data.

- **Published Data** has been published and archived in a publicly accessible location (e.g. a data repository like [zenodo](https://zenodo.org) or [figshare](https://figshare.com)). We do not recommend uploading this data to the cloud directly, but to instead transform it to Zarr and upload it to the cloud. This ensures that the data is stored in an ARCO format and can be easily accessed by other LEAP members.

- **Original Data** is any dataset that is produced by researchers at LEAP and has not been published yet. The main use case for this data is to share it with other LEAP members and collaborate on it. For original data we support direct uploaded to the cloud. *Be aware that original data could change rapidly as the data producer is iterating on their code*. We encourage all datasets to be archived and published before using them in scientific publications.

Step one is always to check whether data already exists in the [LEAP data catalog](guide.data.catalog), in which case access is simply a matter of pasting a code snippet.

(guides.data.ingestion)=

## Ingesting Datasets into Cloud Storage

If you do not find your dataset in the data catalog we can ingest it. Data ingestion in this context means that we have a programmatic way to download and transform data into [Analysis-Ready Cloud-Optimized (ARCO)](reference.arco) formats in a reproducible way, so that the dataset is available for the LEAP community and beyond (see [](reference.infrastructure.buckets) for who can access which resource).

Based on the 3 [types of data](explanation.data-policy.types) we host in the [](explanation.architecture.data-library) there are different ways of ingesting data:

- Linking an existing (public, egress-free) ARCO dataset to the [](explanation.architecture.catalog)
- Ingesting and transforming data into an ARCO copy on [](reference.infrastructure.buckets).
- (Work in Progress): Creating a [virtual zarr store](https://virtualizarr.readthedocs.io/en/latest/) from existing publicly hosted legacy format data (e.g. netcdf)

The end result should feel indistinguishable to the user (i.e. they just copy and paste a snippet and can immediately get to work [](explanation.data-policies.access))

If you are interested in publishing archival or original data, please refer to our How-To guide on [data ingestion](guides.data.ingestion_pipeline).

(guide.data.upload_manual_deprecated)=

## Manually uploading data into Cloud Storage

:::\{warning}
There might be special situations where it is beneficial/necessary to upload data to the [](reference.infrastructure.buckets) but we generally encourage data ingestion to the [](reference.infrastructure.osn_pod) due to the public access and reduced running cost. See above for instructions.
:::

We discourage manually moving datasets to our cloud storage as much as possible since it is hard to reproduce these datasets at a future point (if e.g. the dataset maintainer has moved on to a different position) (see [](explanation.data-policy.reproducibility). We encourage you to try out the methods above, but if these should not work for some reason (and you were not able to find a solution with the [](support.data_compute_team)), you should try the methods below. We will always [prioritize unblocking your work](explanation.code-policy.dont-let-perfect-be-the-enemy-of-good).

The below solutions fundamentally rely on the data being 'pushed' to the [](reference.infrastructure.buckets) which usually requires intervention on part of the [](explanation.data-policy.roles.data-expert). This stands in contrast to e.g. data ingestion where the [](explanation.data-policy.roles.data-expert) only has to work on the recipe creation and the data is 'pulled' in a reproducible way. For more information see [](explanation.data-policy).

<!-- Fundamentally the 'pushing' of datasets relies on two components:

- Setting up permissions so that you can read/write to the [](reference.infrastructure.buckets) - Several methods to get permissions are described in [](reference.authentication).
- Initiating a data transfer from your 'local' machine (e.g. your laptop, a server, or HPC Cluster). You can check on some methods below. -->

## Tools for Cloud Access

There are many tools available to interact with cloud object storage. We currently have basic operations documented for two tools:

- [fsspec](https://filesystem-spec.readthedocs.io/en/latest/) (and its submodules [gcsfs](https://gcsfs.readthedocs.io/en/latest/) and [s3fs](https://s3fs.readthedocs.io/en/latest/)) which provide filesystem-like access to local, remote, and embedded file systems from within a python session. Fsspec is also used by xarray under the hood.
  `fsspec` can be used to transfer small amounts of data to google cloud storage or OSN. If you have more than a few hundred MB's, it is worth using Rclone.
  `fsspec` should be installed by default on the **Base Pangeo Notebook** environment on the Jupyter-Hub. If you are working locally, you can install it with `pip or conda/mamba`. ex: `pip install fsspec`.

- [rclone](https://rclone.org/) which provides a Command Line Interface to many different storage backends.

:::\{admonition} Note on rclone documentation
:class: tip, dropdown
Rclone is a very extensive and powerful tool, but with its many options it can be overwhelming (at least it was for Julius) at the beginning. We will only demonstrate essential options here, for more details see the [docs](https://rclone.org/). If however instructions here are not working for your specific use case, please reach out so we can improve the docs.
:::

Using [`fsspec.open`](https://filesystem-spec.readthedocs.io/en/latest/api.html#fsspec.open) works similarly to the python builtin [`open`](https://docs.python.org/3/library/functions.html#open)

```python
with fsspec.open("gs://leap-scratch/funky-user/test.txt", mode="w") as f:
    f.write("hello world")
```

(data.config-files)=

## Authentication

Unless a given cloud bucket allows anonymous access or is preauthenticated within your environment, you will need to authenticate with a key/secret pair. The [LEAP-Pangeo owned buckets](reference.infrastructure.buckets) are pre-authenticated on the Hub but working with them externally requires configuration. **Note:** If you are accessing non-public cloud data from the JupyterHub (not published by LEAP), you will also have to follow this process.

:::\{admonition} Always handle credentials with care!
:class: warning
Always handle secrets with care. Do not store them in plain text that is visible to others (e.g. in a notebook cell that is pushed to a public github repository). See [](guide.code.secrets) for more instructions on how to keep secrets safe.
:::

We recommend to store your secrets in one of the following configuration files (which will be used in the following example to read and write data):

`````{tab-set}
````{tab-item} Fsspec
Fsspec supports named [](aws profiles) in a credentials files. You can create one via Generate an aws credential file via the [aws CLI](https://docs.aws.amazon.com/cli/v1/userguide/cli-configure-files.html#cli-configure-files-examples)(installed on the hub by defaule):

```shell
aws configure --profile <pick_a_name>
```
Pick a sensible name for your profile, particularly if you are working with multiple profiles and buckets.

The file `~/.aws/credentials` then contains your key/secret similar to this:

```
[<the_profile_name_you_picked>]
aws_access_key_id = ***
aws_secret_access_key = ***
```
````

````{tab-item} Rclone
Rclone has its own [configuration file format](https://rclone.org/docs/#config-config-file) where you can specify the key and secret (and many other settings) in a similar fashion (note the missing `aws_` though!).

We recommend setting up the config file (show the default location with `rclone config file`) by hand to look something like this:

```
[<remote_name>]
... # other values
access_key_id = XXX
secret_access_key = XXX
```

You can have multiple 'remotes' in this file for different cloud buckets.

For the [](reference.infrastructure.osn_pod) use this remote definition:

```
[osn]
type = s3
provider = Ceph
endpoint = https://nyu1.osn.mghpcc.org
access_key_id = XXX
secret_access_key = XXX
```

````
`````

:::\{warning}
Ideally we want to store these secrets only in one central location. The natural place for these seems to be in an [AWS cli profiles](https://docs.aws.amazon.com/cli/v1/userguide/cli-configure-files.html#cli-configure-files-format), which can also be used for fsspec. There however seem to be multiple issues ([here](https://forum.rclone.org/t/shared-credentials-file-is-not-recognised/46993)) around this feature in rclone, and so far we have not succeeded in using AWS profiles in rclone.
According to those issues we can only make the aws profiles (or [source profiles?](https://forum.rclone.org/t/s3-profile-failing-when-explicit-s3-endpoint-is-present/36063/4?u=jbusecke), anyways the credentials part of it) work if we define one config file per remote [and use the 'default' profile](https://forum.rclone.org/t/shared-credentials-file-is-not-recognised/46993/2?u=jbusecke)which presumably breaks compatibility with fsspec, and also does not work at all right now. So at the moment we will have to keep the credentials in two separate spots 🤷‍♂️. **Please make sure to apply proper caution when [handling secrets](guide.code.secrets) for each config files that stores secrets in plain text!**
:::

:::\{note}
You can find more great documentation, specifically on how to use OSN resources, in [this section](https://hytest-org.github.io/hytest/essential_reading/DataSources/Data_S3.html#credentialed-access) of the [HyTEST Docs](https://hytest-org.github.io/hytest/doc/About.html#)
:::

(hub.data.list)=

## Reading and Writing Data

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

For **authenticated access** you need to pass additional arguments. In this case (for the m2lines OSN pod) we pass a custom endpoint and an [aws profile](data.config-files):
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

To inspect a bucket you can use clone with the profile ('remote' in rclone terminology) set up [above](data.config-files):

```shell
rclone ls <remote_name>:bucket-name/funky-user
```

````
`````

### Choosing a location - OSN vs GCS

Details on both OSN and GCS cloud storage buckets can be found in the [infrastructure guide](reference.infrastructure.osn_pod).

For some general guidelines on choosing:

GCS:
\- You want to move data from your Jupyter-Hub home directory to the cloud.
\- You don't need the data to be accessed outside of the Jupyter-Hub.
\- This data is a work-in-progress and might be regenerated or modified.

OSN:
\- You want your data to be publicly accessible outside of the Jupyter-Hub.
\- You need to move data from your Jupyter-Hub home directory to more persistent storage.
\- You data does not fit into the Zarr model.

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

You can move directories from a local computer to cloud storage with rclone (make sure you are properly [authenticated](data.config-files)):

```shell
rclone copy path/to/local/dir/ <remote_name>:<bucket-name>/funky-user/some-directory
```

You can also move data between cloud buckets using rclone

```shell
rclone copy \
 <remote_name_a>:<bucket-name>/funky-user/some-directory\
 <remote_name_b>:<bucket-name>/funky-user/some-directory
```
:::{admonition} Copying single files
:class: note
To copy single files with rclone use the [copyto command](https://rclone.org/commands/rclone_copyto/) or copy the containing folder and use the `--include` or `--exclude` flags to select the file to copy.  
:::

:::{note}
Copying with rclone will stream the data from the source to your computer and back to the target, and thus transfer speed is likely limited by the internet connection of your local machine.
:::

````
`````

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
