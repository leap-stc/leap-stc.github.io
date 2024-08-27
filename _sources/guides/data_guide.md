(guide.data)=

# Data Guide

Data is fundamental to most people's work at LEAP. This guide describes best practices how to find, transfer, ingest, and catalog data.

## Discovering Dataset

You want to have a specific dataset to explore or analyze? There is a good chance that somebody else at LEAP has already worked with the data! So the first thing to look for data should always be a visit to the [](reference.infrastructure.catalog).

## Working with Data in Cloud Object Storage

Data and files work differently in the cloud.
To help onboard you to this new way of working, we have written a guide to Files and Data in the Cloud:

- [2i2c Docs: Data and Filesystem](https://docs.2i2c.org/user/topics/data/filesystem/)

We recommend you read this thoroughly, especially the part about Git and GitHub. LEAP provides several [cloud buckets](reference.infrastructure.buckets), and the following steps illustrate how to work with data in object storage as opposed to a filesystem.

(hub.data.list)=

### Inspecting contents of the bucket

We recommend using [gcsfs](https://gcsfs.readthedocs.io/en/latest/) or [fsspec](https://filesystem-spec.readthedocs.io/en/latest/) which provide a filesytem-like interface for python.

You can e.g. list the contents of your personal folder with

```python
import gcsfs

fs = gcsfs.GCSFileSystem()  # equivalent to fsspec.fs('gs')
fs.ls("leap-persistent/funky-user")
```

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

:::\{note}
Note that providing the url starting with `gs://...` is assumes that you have appropriate credentials set up in your environment to read/write to that bucket. On the hub these are already set up for you to work with the [](reference.infrastructure.buckets), but if you are trying to interact with non-public buckets you need to authenticate yourself. Check out the sections [below](guide.data.upload_manual) to see an example how to do that.
:::

You can also write other files directly to the bucket by using [`fsspec.open`](https://filesystem-spec.readthedocs.io/en/latest/api.html#fsspec.open) similarly to the python builtin [`open`](https://docs.python.org/3/library/functions.html#open)

```python
with fsspec.open("gs://leap-scratch/funky-user/test.txt", mode="w") as f:
    f.write("hello world")
```

Another example of a rountrip save and load with numpy:

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

:::\{warning}
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

We have additional requirements for the data ingestion to make the process sustainable and scalable:

- Process needs to be reproducible, e.g. when we want to reingest data to a different storage location
- Separation of concerns: The person who knows the dataset (the 'data expert') is in the unique position to encode their knowledge about the dataset into the recipe, but they should not be concerned with the details of how to execute it and where the data is ultimate stored. This is the responsibility of the Data and Compute team.

The way we achieve this is to base our ingestion on [Pangeo Forge recipes](https://pangeo-forge.readthedocs.io/en/latest/composition/index.html#recipe-composition). For clearer organization each dataset the recipe should reside in its own repository under the `leap-stc` github organization. Each of these repositories will be called a 'feedstock', which contains additional metadata files (you can read more in the [Pangeo Forge docs](https://pangeo-forge.readthedocs.io/en/latest/deployment/feedstocks.html#from-recipe-to-feedstock)).

#### How to get new data ingested

To start ingesting a dataset follow these steps:

1. Let the LEAP community and the Data and Computation Team know about this new dataset. We gather all ingestion requests in our ['leap-stc/data_management' issue tracker](https://github.com/leap-stc/data-management/issues). You should check existing issues with the tag ['dataset'](https://github.com/leap-stc/data-management/issues?q=is%3Aissue+is%3Aopen+label%3Adataset) to see if somebody else might have already requested this particular dataset. If that is not the case you can add a new [dataset_request](https://github.com/leap-stc/data-management/issues/new?assignees=&labels=dataset&projects=&template=new_dataset.yaml&title=New+Dataset+%5BDataset+Name%5D). Making these request in a central location enables others to see which datasets are currently being ingested and what the status is.
1. Use our [feedstock template](https://github.com/leap-stc/LEAP_template_feedstock) to create a feedstock repostory by following instructions in the README to get you started with either one of the above.
1. If issues arise please reach out to the [](support.data_compute_team)

:::\{note}
This does currently not provide a solution to handle datasets that have been produced by you (as e.g. part of a publication). We are working on formalizing a workflow for this type of data. Please reach out to the [](support.data_compute_team) if you have data that you would like to publish. See [](explanation.data-policy.types) for more.
:::

(guide.data.upload_manual)=

### Manually uploading/downloading data to cloud buckets

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
