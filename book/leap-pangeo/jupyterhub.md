# LEAP-Pangeo JupyterHub

Our team has a cloud-based [JupyterHub](https://jupyter.org/hub).
For information who can access the hub with which privileges, please refer to
{doc}`/policies/users_roles`

|                       |                                                                                   |
| --------------------- | --------------------------------------------------------------------------------- |
| **Hub Address**       | <https://leap.2i2c.cloud/>                                                        |
| **Hub Location**      | [Google Cloud `us-central1`](https://cloud.google.com/compute/docs/regions-zones) |
| **Hub Operator**      | [2i2c](https://2i2c.org/)                                                         |
| **Hub Configuration** | <https://github.com/2i2c-org/infrastructure/tree/master/config/clusters/leap>     |

This document goes over the primary technical details of the JupyterHub.

- For a quick tutorial on basic usage, please see [Getting Started](tutorial.md).
- To get an in-depth overview of the LEAP Pangeo Architecture and how the JupyterHub fits into it, please see the [Architecture](architecture.md) page.

(jupyterhub.software_env)=

## The Software Environment

The software environment you encounter on the Hub is based upon [docker images](https://www.digitalocean.com/community/tutorials/the-docker-ecosystem-an-introduction-to-common-components) which you can run on other machines (like your laptop or an HPC cluster) for better reproducibility.

Upon start up you can choose between

- A list of preselected images
- The option of passing a custom docker image via the `"Other..."` option.

### Preselected Images

LEAP-Pangeo uses several full-featured, up-to-date Python environments maintained by Pangeo. You can read all about them at the following URL:

- <https://github.com/pangeo-data/pangeo-docker-images/>

There are separate images for pytorch and tensorflow which are available in a drop-down panel when starting up your server.
The Hub contains a specific version of the image which can be found [here](https://github.com/2i2c-org/infrastructure/blob/master/config/clusters/leap/common.values.yaml).

For example, at the time of writing, the version of `pangeo-notebook` is `2022.05.10`.
A complete list of all packages installed in this environment is located at:

- <https://github.com/pangeo-data/pangeo-docker-images/blob/2022.05.10/pangeo-notebook/packages.txt>

:::\{attention}
We regularly update the version of the images provided in the drop-down menu.

To ensure full reproducibility you should save the full info of the image you worked with (this is stored in the environment variable `JUPYTER_IMAGE_SPEC`) with your work. You could for example print the following in the first cell of a notebook:

```python
import os

print(os.environ["JUPYTER_IMAGE_SPEC"])
```

You can then use that string with the [custom images](hub.image.custom) to reproduce your work with exactly the same environment.
:::

(hub.image.custom)=

### Custom Images

If you select the `Image > Other...` Option during [server login](hub:server:login) you can paste an arbitrary reference in the form of `docker_registry/organization/image_name:image_version`. As an example we can get the `2023.05.08` version of the pangeo tensorflow notebook by pasting `quay.io/pangeo/ml-notebook:2023.05.08`.

If you want to build your own docker image for your project, take a look at [this template](https://github.com/2i2c-org/hub-user-image-template) and the instructions to learn how to use [repo2docker](https://github.com/jupyterhub/repo2docker) to set up CI workflows to automatically build docker images from your repository.

### Installing additonal packages

You can install additional packages using `pip` and `conda`.
However, these will disappear when your server shuts down.

For a more permanent solution we recommend building project specific dockerfiles and using those as [custom images](hub.image.custom).

## Files and Data

Data and files work differently in the cloud.
To help onboard you to this new way of working, we have written a guide to Files and Data in the Cloud:

- [2i2c Docs: Data and Filesystem](https://docs.2i2c.org/user/topics/data/filesystem/)

We recommend you read this thoroughly, especially the part about Git and GitHub.

(hub.guide.data.user_dir)=

### Your User Directory

When you open your hub, you can navigate to the "File Browser" and see all the files in your User Directory
<img width="442" alt="image" src="https://github.com/leap-stc/leap-stc.github.io/assets/14314623/3ba6b45a-a077-4824-b0ec-9c111af50c33">

Your User Directory behaves very similar to a filestystem on your computer. If you save a file from a notebook, you will see it appear in the File Browser (you might have to wait a few seconds or press refresh) and you can use a terminal to navigate the terminal as you would on a UNIX machine:

<img width="357" alt="image" src="https://github.com/leap-stc/leap-stc.github.io/assets/14314623/a84c12e2-9f8a-4de1-a3e3-feff1bf59061">

:::\{note}
As shown in the picture above, every user will see `'/home/jovyan'` as their root directory. This is different from many HPC accounts where your home directory will point to a directory with your username. But the functionality is similar. These are *your own files* and they cannot be seen/modified by other users (except admins).
:::

The primary purpose of this directory is to store small files, like github repositories and other code.

:::\{warning}
Please do not store large files in your user directory `/home/jovyan`. Your home directory is intended only for notebooks, analysis scripts, and small datasets (\< 1 GB). It is not an appropriate place to store large datasets. Unlike the cloud buckets, these directories all use the same underlying storage so if a single user fills up the space, the Hub crashes for everyone. To accommodate the expanding LEAP community, the data and compute team has instituted a storage quota. We recommend users use less than 25GB and maintain a hard limit of 50GB. **Users who persistently violate the hard limit may temporarily get reduced cloud access**.

To check how much space you are using in your home directory open a terminal window on the hub and run `du -h --max-depth=1 ~/ | sort -h`.

If you want to save larger files for your work use our [](hub.data.buckets) and consult our [Hub Data Guide](guide.hub.data)
:::

(hub.data.buckets)=

### LEAP-Pangeo Cloud Storage Buckets

LEAP-Pangeo provides users two cloud buckets to store data

- `gs://leap-scratch/` - Temporary Storage deleted after 7 days. Use this bucket for testing and storing large intermediate results. [More info](https://docs.2i2c.org/user/topics/data/cloud/#scratch-bucket)
- `gs://leap-persistent/` - Persistent Storage. Use this bucket for storing results you want to share with other members.
- `gs://leap-persistent-ro/` - Persistent Storage with read-only access for most users. To upload data to this bucket you need to use [this](hub.data.upload_hpc) method below.

Files stored on each of those buckets can be accessed by any LEAP member, so be concious in the way you use these.

- **Do not put sensitive information (passwords, keys, personal data) into these buckets!**
- **When writing to buckets only ever write to your personal folder!** Your personal folder is a combination of the bucketname and your github username (e.g. \`gs://leap-persistent/funky-user/').

(hub.data.list)=

#### Inspecting contents of the bucket

We recommend using [gcsfs](https://gcsfs.readthedocs.io/en/latest/) or [fsspec](https://filesystem-spec.readthedocs.io/en/latest/) which provide a filesytem-like interface for python.

You can e.g. list the contents of your personal folder with

```python
import gcsfs

fs = gcsfs.GCSFileSystem()  # equivalent to fsspec.fs('gs')
fs.ls("leap-persistent/funky-user")
```

(hub.data.read_write)=

#### Basic writing to and reading from cloud buckets

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
Note that providing the url starting with `gs://...` is assumes that you have appropriate credentials set up in your environment to read/write to that bucket. On the hub these are already set up for you to work with the [](hub.data.buckets), but if you are trying to interact with non-public buckets you need to authenticate yourself. Check out the sections [below](hub.guide.data.upload_manual) to see an example how to do that.
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

#### Deleting from cloud buckets

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
