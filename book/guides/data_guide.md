(guide.data)=

# Data Guide

Data is fundamental to most people's work at LEAP. This guide describes best practices for how to find, read, write, transfer, ingest, and catalog data. It is meant as a broad overview to common workflows and institutional policies. We categorize LEAP-relevant use of data into three categories: 
- Using the data that LEAP has catalogued and made accessible in ARCO format. This is done via our [Data Catalog](reference.infrastructure.catalog).
- Working with data that exists in the cloud. This is in conjunction with the [JupyterHub platform](reference.infrastructure.hub) as an alternative to other data science workflows. 
- Publishing new scientific data for consumption by other parties. This includes ingesting data into [public cloud buckets](reference.infrastructure.buckets).

To learn how to work with existing data alraedy in the cloud, please consult [](guide.data.working). To learn how to publish or move your own data into cloud storage, please consult [](guide.data.transfer).

## Discovering Dataset

You want to have a specific dataset to explore or analyze? There is a good chance that somebody else at LEAP has already worked with the data! So the first thing to look for data should always be a visit to the [LEAP Data Catalog](catalog.leap.columbia.edu). This is a repository of data sets published by the LEAP community in collaboration with the Data and Compute Team. The home page will immediately show a list of which datasets are included. Every dataset has a brief description, provides a simple code snippet for loading the data into Python, and links to the original feedstock from which the data was ingested. The term ["feedstock"](https://pangeo-forge.readthedocs.io/en/latest/deployment/feedstocks.html) is inherited from the Pangeo Forge project, and basically refers to the code repository defining the data pipeline. Feedstocks allow curious users to trace back towards the original data source for transparency and reproducibility. 

The basic requirements for loading the data are the following packages, which are automatically accessible to any user of the JupyterHub platform. But if you wish to load the data on your machine, then you must ensure your python environment has the following pacakges:
```
xarray
requests
aiohttp
dask
zarr
fsspec
```

### Working with Data in Cloud Object Storage

Data and files work differently in the cloud.
To help onboard you to this new way of working, we have written a guide to Files and Data in the Cloud:

- [2i2c Docs: Data and Filesystem](https://docs.2i2c.org/user/topics/data/filesystem/)

We recommend you read this thoroughly, especially the part about Git and GitHub. It provides
- an overview of the different directories visible upon login to the JupyterHub and what they are meant for. 
- how to authenticate to Github from the Hub and integrate it with your Hub workflow. 
- how users should share files with each other. 
LEAP provides several [cloud buckets](reference.infrastructure.buckets), and the following steps illustrate how to work with data in object storage as opposed to a filesystem.

### Tools

There are many tools available to interact with cloud object storage. We currently have basic operations documented for two tools:

- [fsspec](https://filesystem-spec.readthedocs.io/en/latest/) (and its submodules [gcsfs](https://gcsfs.readthedocs.io/en/latest/) and [s3fs](https://s3fs.readthedocs.io/en/latest/)) which provide filesystem-like access to local, remote, and embedded file systems from within a python session. Fsspec is also used by xarray under the hood.

- [rclone](https://rclone.org/) which provides a Command Line Interface to many different storage backends.

:::{admonition} Note on rclone documentation
:class: tip, dropdown
Rclone is a very extensive and powerful tool, but with its many options it can be overwhelming (at least it was for Julius) at the beginning. We will only demonstrate essential options here, for more details see the [docs](https://rclone.org/). If however instructions here are not working for your specific use case, please reach out so we can improve the docs.
:::

(data.config-files)=

#### Configuration for Authenticated Access

Unless a given cloud bucket allows anonymous access or is preauthenticated within your environment (like it is the case for some of the [LEAP-Pangeo owned buckets](reference.infrastructure.buckets)) you will need to authenticate with a key/secret pair.

:::{admonition} Always Handle credentials with care!
:class: warning
Always handle secrets with care. Do not store them in plain text that is visible to others (e.g. in a notebook cell that is pushed to a public github repository). See [](guide.code.secrets) for more instructions on how to keep secrets safe.
:::

We recommend to store your secrets in one of the following configuration files (which will be used in the following example to read and write data):
s

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

For the [](reference.infrastructrue.osn_pod) use this remote definition:

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

:::{warning}
Ideally we want to store these secrets only in one central location. The natural place for these seems to be in an [AWS cli profiles](https://docs.aws.amazon.com/cli/v1/userguide/cli-configure-files.html#cli-configure-files-format), which can also be used for fsspec. There however seem to be multiple issues ([here](https://forum.rclone.org/t/shared-credentials-file-is-not-recognised/46993)) around this feature in rclone, and so far we have not succeeded in using AWS profiles in rclone.
According to those issues we can only make the aws profiles (or [source profiles?](https://forum.rclone.org/t/s3-profile-failing-when-explicit-s3-endpoint-is-present/36063/4?u=jbusecke), anyways the credentials part of it) work if we define one config file per remote [and use the 'default' profile](https://forum.rclone.org/t/shared-credentials-file-is-not-recognised/46993/2?u=jbusecke)which presumably breaks compatibility with fsspec, and also does not work at all right now. So at the moment we will have to keep the credentials in two separate spots 🤷‍♂️. **Please make sure to apply proper caution when [handling secrets](guide.code.secrets) for each config files that stores secrets in plain text!**
:::

:::{note}
You can find more great documentation, specifically on how to use OSN resources, in [this section](https://hytest-org.github.io/hytest/essential_reading/DataSources/Data_S3.html#credentialed-access) of the [HyTEST Docs](https://hytest-org.github.io/hytest/doc/About.html#)
:::

(hub.data.setup)=

### 

(hub.data.list)=

### Inspecting contents of the bucket

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

### Moving Data

`````{tab-set}
````{tab-item} Fsspec
🚧
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

(hub.data.read_write)=

### Basic writing to and reading from cloud buckets

We do not recommend uploading large files (e.g. netcdf) directly to the bucket. Instead we recommend to write data as ARCO (Analysis-Ready Cloud-Optimized) formats like [zarr](https://zarr.dev)(for n-dimensional arrays) and [parquet](https://parquet.apache.org)(for tabular data) (read more [here](https://ieeexplore.ieee.org/document/9354557) why we recommend ARCO formats).

If you work with xarray Datasets switching the storage format is as easy as swapping out a single line when reading/writing data:

Xarray provides a method to stream results of a computation to zarr

```python
ds = ...
ds_processed = ds.mean(...).resample(...)
user_path = "gs://leap-scratch/funky-user"  # 👀 make sure to prepend `gs://` to the path or xarray will interpret this as a local path
store_name = "processed_store.zarr"
ds_processed.to_zarr(f"{user_path}/{store_name}")
```

This will write a zarr store to the scratch bucket.

You can read it back into an xarray dataset with this snippet:

```python
import xarray as xr

ds = xr.open_dataset(
    "gs://leap-scratch/funky-user/processed_store.zarr", engine="zarr", chunks={}
)  #
```

... and you can give this to any other registered LEAP user and they can load it exactly like you can!

:::{note}
Note that providing the url starting with `gs://...` is assumes that you have appropriate credentials set up in your environment to read/write to that bucket. On the hub these are already set up for you to work with the [](reference.infrastructure.buckets), but if you are trying to interact with non-public buckets you need to authenticate yourself. Check out [](data.config-files) to see an example how to do that.
:::

You can also write other files directly to the bucket by using [`fsspec.open`](https://filesystem-spec.readthedocs.io/en/latest/api.html#fsspec.open) similarly to the python builtin [`open`](https://docs.python.org/3/library/functions.html#open)

```python
with fsspec.open("gs://leap-scratch/funky-user/test.txt", mode="w") as f:
    f.write("hello world")
```

Another example of a roundtrip save and load with numpy:

```python
import numpy as np
import fsspec

arr = np.array([1, 2, 4])
arr
```

```
array([1, 2, 4])
```

```python
with fsspec.open("gs://leap-scratch/funky-user/arr_test.npy", mode="wb") as f:
    np.save(f, arr)

with fsspec.open("gs://leap-scratch/jbusecke/arr_test.npy", mode="rb") as f:
    arr_reloaded = np.load(f)

arr_reloaded
```

```
array([1, 2, 4])
```

> Make sure to specify `mode='rb'` or `move='wb'` for binary files.

### Deleting from cloud buckets

:::{warning}
Depending on which cloud bucket you are working, make sure to double check which files you are deleting by [inspecting the contents](hub.data.list) and only working in a subdirectory with your username (e.g. `gs://<leap-bucket>/<your-username>/some/project/structure`.
:::

You can remove single files by using a gcsfs/fsspec filessytem as above

```python
import gcsfs

fs = gcsfs.GCSFileSystem()  # equivalent to fsspec.fs('gs')
fs.rm("leap-persistent/funky-user/file_to_delete.nc")
```

If you want to remove zarr stores (which are an 'exploded' data format, and thus represented by a folder structure) you have to recursively delete the store.

```python
fs.rm("leap-scratch/funky-user/processed_store.zarr", recursive=True)
```

## Transfering Data into Cloud Storage

We distinguish between two primary *types* of data to upload: "Original" and "Published" data.

- **Published Data** has been published and archived in a publically accessible location (e.g. a data repository like [zenodo](https://zenodo.org) or [figshare](https://figshare.com)). We do not recommend uploading this data to the cloud directly, but instead use [Pangeo Forge](https://pangeo-forge.readthedocs.io/en/latest/) to transform and upload it to the cloud. This ensures that the data is stored in an ARCO format and can be easily accessed by other LEAP members.
- **Original Data** is any dataset that is produced by researchers at LEAP and has not been published yet. The main use case for this data is to share it with other LEAP members and collaborate on it. For original data we support direct uploaded to the cloud. *Be aware that original data could change rapidly as the data producer is iterating on their code*. We encourage all datasets to be archived and published before using them in scientific publications.

(guides.data.ingestion)=

### Ingesting Datasets into Cloud Storage

If you do not find your dataset in the data catalog we can ingest it. Data ingestion in this context means that we have a programatic way to download and transform data into [Analysis-Ready Cloud-Optimized (ARCO)](reference.arco) formats in a reproducible way, so that the dataset is available for the LEAP community and beyond (see [](reference.infrastructure.buckets) for who can access which resource).

Based on the 3 [types of data](explanation.data-policy.types) we host in the [](explanation.architecture.data-library) there are different ways of ingesting data:

- Linking an existing (public, egress-free) ARCO dataset to the [](explanation.architecture.catalog)
- Ingesting and transforming data into an ARCO copy on [](reference.infrastructure.buckets).
- (Work in Progress): Creating a virtual zarr store from existing publically hosted legacy format data (e.g. netcdf)

The end result should feel indistingushable to the user (i.e. they just copy and paste a snippet and can immediately get to work [](explanation.data-policies.access))

If you are interested in publishing archival or original data, please refer to our How-To guide on [data ingestion](guides.data.ingestion_pipeline).

(guide.data.upload_manual_deprecated)=

### Manually uploading/downloading data to cloud buckets (deprecated)

:::{warning}
This section of the docs is just retained for completeness. There might be special situations where it is beneficial/necessary to upload data to the [](reference.infrastructure.buckets) but we generally encourage data ingestion to the [](reference.infrastructrue.osn_pod) due to the public access and reduced running cost. See above for instructions.
:::

We discourage manually moving datasets to our cloud storage as much as possible since it is hard to reproduce these datasets at a future point (if e.g. the dataset maintainer has moved on to a different position) (see [](explanation.data-policy.reproducibility). We encourage you to try out the methods above, but if these should not work for some reason (and you were not able to find a solution with the [](support.data_compute_team)), you should try the methods below. We will always [prioritize unblocking your work](explanation.code-policy.dont-let-perfect-be-the-enemy-of-good).

The below solutions fundamentally rely on the data being 'pushed' to the [](reference.infrastructure.buckets) which usually requires intervention on part of the [](explanation.data-policy.roles.data-expert). This stands in contrast to e.g. data ingestion via [](reference.pangeo-forge) where the [](explanation.data-policy.roles.data-expert) only has to work on the recipe creation and the data is 'pulled' in a reproducible way. For more information see [](explanation.data-policy).

Fundamentally the 'pushing' of datasets relies on two components:

- Setting up permissions so that you can read/write to the [](reference.infrastructure.buckets) - Several methods to get permissions are described in [](reference.authentication).
- Initiating a data transfer from your 'local' machine (e.g. your laptop, a server, or HPC Cluster). You can check on some methods below.

#### Upload medium sized original data from your local machine

For medium sized datasets, that can be uploaded within an hour, you can use a temporary access token generated on the JupyterHub to upload data to the cloud.

- Set up a new environment on your local machine (e.g. laptop)

```shell
mamba env create --name leap_pangeo_transfer python=3.9 google-auth gcsfs jupyterlab xarray zarr dask
```

> Add any other dependencies (e.g. netcdf4) that you need to read your data at the end of the line

- Activate the environment

```shell
conda activate leap_pangeo_transfer
```

and set up a jupyter notbook (or a pure python script) that loads your data in as few xarray datasets as possible. For instance, if you have one dataset that consists of many files split in time, you should set your notebook up to read all the files using xarray into a single dataset, and then try to write out a small part of the dataset to a zarr store.

Now [generate a temporary token](reference.authentication.temp_token) and copy the resulting token into a plain text file `token.txt` in a convenient location on your **local machine**.

- Now start a JupyterLab notebook and paste the following code into a cell:

```python
import gcsfs
import xarray as xr
from google.cloud import storage
from google.oauth2.credentials import Credentials

# import an access token
# - option 1: read an access token from a file
with open("path/to/your/token.txt") as f:
    access_token = f.read().strip()

# setup a storage client using credentials
credentials = Credentials(access_token)
fs = gcsfs.GCSFileSystem(token=credentials)
```

> Make sure to replace the `path/to/your/token.txt` with the actual path to your token file.

Try to write a small dataset to the cloud:

```python
ds = xr.DataArray([1]).to_dataset(name="test")
mapper = fs.get_mapper(
    "gs://leap-scratch/<your_username>/test_offsite_upload.zarr"
)  # This additional step is necessary to have the correct authentication set
ds.to_zarr(mapper)
```

> Replace `<your_username>` with your actual username on the hub.

- Make sure that you can read the test dataset from within the hub (go back to [Basic writing to and reading from cloud buckets](hub.data.read_write)).

- Now the last step is to paste the code to load your actual dataset into the notebook and use `.to_zarr` to upload it.

> Make sure to give the store a meaningful name, and raise an issue in the [data-management repo](https://github.com/leap-stc/data-management/issues) to get the dataset added to the LEAP Data Library.

> Make sure to use a different bucket than `leap-scratch`, since that will be deleted every 7 days! For more info refer to the available [storage buckets](reference.infrastructure.buckets).

(hub.data.upload_hpc)=

#### Uploading large original data from an HPC system (no browser access on the system available)

A commong scenario is the following: A researcher/student has run a simulation on a High Performance Computer (HPC) at their institution, but now wants to collaboratively work on the analysis or train a machine learning model with this data. For this they need to upload it to the cloud storage.

The following steps will guide you through the steps needed to authenticate and upload data to the cloud, but might have to be slightly modified depending on the actual setup of the users HPC.

**Conversion Script/Notebook**

In most cases you do not just want to upload the data in its current form (e.g. many netcdf files).

<!-- TODO: Add an example of why this is bad for performance -->

Instead we will load the data into an [`xarray.Dataset`](https://docs.xarray.dev/en/stable/generated/xarray.Dataset.html) and then write that Dataset object directly to a zarr store in the cloud. For this you need a python environment with `xarray, gcsfs, zarr` installed (you might need additional dependencies for your particular use case).

1. Spend some time to set up a python script/jupyter notebook on the HPC system that opens your files and combines them in to one or more xarray.Datasets (combine as many files as sensible into a single dataset). Make sure that your data is lazily loaded and the `Dataset.data` is a [dask array](https://docs.dask.org/en/stable/array.html)

1. Check your dataset:

   - Check that the metadata is correct.
   - Check that all the variables/dimensions are in the dataset
   - Check the dask chunksize. A general rule is to aim for around 100MB size, but the size and structure of chunking that is optimal depends heavily on the later use case.

   <!-- Some more info on chunking? -->

1. Try to write out a subset of the data locally by calling the [`.to_zarr`](https://docs.xarray.dev/en/stable/generated/xarray.Dataset.to_zarr.html) method on the dataset.

<!-- TODO: Warn not to write out the full dataset -->

Once that works we can move on to the authentication.

**Upload Prerequisites**

Before we are able to set up authentication we need to make sure our HPC and local computer (required) are set up correctly.

- You have to be signed up to LEAP's [Google Groups](reference.authentication.google_groups).
- Make sure to install the [Google Cloud SDK](https://cloud.google.com/sdk/docs/install) in both your <span style="color:#9301B4">HPC environment</span>, and your local computer that can open a web browser (e.g. your laptop).

**Steps**
Steps executed on your <span style="color:#22B401">"local" computer (e.g. laptop)</span> will be colored in green and steps on your <span style="color:#9301B4">"remote" computer (e.g. HPC)</span> in purple.

1. SSH into the HPC

1. <span style="color:#9301B4">Check that you have an internet connection with `ping www.google.com`</span>

1. <span style="color:#9301B4">Request no browser authentication: </span>

   ```
   gcloud auth application-default login --scopes=https://www.googleapis.com/auth/devstorage.read_write,https://www.googleapis.com/auth/iam.test --no-browser
   ```

   > 🚨 It is very important to include the `--scopes=` argument for security reasons. Do not run this command without it!

1. Follow the onscreen prompt and paste the command into a terminal on your local machine.

1. <span style="color:#22B401">This will open a browser window. Authenticate with the gmail account that was added to the google group. </span>

1. <span style="color:#22B401">Go back to the terminal and follow the onscreen instructions.</span> Copy the text from the command line and <span style="color:#9301B4">paste the command in the open dialog on the remote machine.</span>

1. <span style="color:#9301B4">Make sure to note the path to the auth json!</span> It will be something like `.../.config/gcloud/....json`.</span>

Now you are have everything you need to authenticate.

Lets verify that you can write a small dummy dataset to the cloud. In your notebook/script run the following (make sure to replace the filename and your username as instructed).

Your dataset should now be available for all LEAP members 🎉🚀

```python
import xarray as xr
import gcsfs
import json

with open(
    "your_auth_file.json"
) as f:  # 🚨 make sure to enter the `.json` file from step 7
    token = json.load(f)

# test write a small dummy xarray dataset to zarr
ds = xr.DataArray([1, 4, 6]).to_dataset(name="data")
# Once you have confirmed

fs = gcsfs.GCSFileSystem(token=token)
mapper = fs.get_mapper(
    "gs://leap-persistent/<username>/testing/demo_write_from_remote.zarr"
)  # 🚨 enter your leap (github) username here
ds.to_zarr(mapper)
```

Now you can repeat the same steps but replace your dataset with the full dataset from above and leave your python code running until the upload has finished. Depending on the internet connection speed and the size of the full dataset, this can take a while.

If you want to see a progress bar, you can wrap the call to `.to_zarr` with a [dask progress bar](https://docs.dask.org/en/stable/diagnostics-local.html#progress-bar)

```python
from dask.diagnostics import ProgressBar

with ProgressBar():
    ds.to_zarr(mapper)
```

Once the data has been uploaded, make sure to erase the `.../.config/gcloud/....json` file from step 7, and ask to be removed from the Google Group.
