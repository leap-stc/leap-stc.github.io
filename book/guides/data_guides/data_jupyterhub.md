(guide.data.working)=
# Working with Data in Cloud Object Storage

Data and files work differently in the cloud.
To help onboard you to this new way of working, we have written a how-to guide to working with Files and Data in the Cloud: 

- [2i2c Docs: Data and Filesystem](https://docs.2i2c.org/user/topics/data/filesystem/)

We recommend you read this thoroughly, especially the part about Git and GitHub. It provides
- an overview of the different directories visible upon login to the JupyterHub and what they are meant for. 
- how to authenticate to Github from the Hub and integrate it with your Hub workflow. 
- how users should share files with each other. 
LEAP provides several [cloud buckets](reference.infrastructure.buckets), and the following steps illustrate how to work with data in object storage as opposed to a filesystem. 

## Working from the LEAP JupyterHub

### How to load Data from the LEAP Catalog
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

### How to read and write data from cloud buckets

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

### How to delete data from cloud buckets

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

## Working from outside the JupyterHub
Although the LEAP Jupyterhub was created to vastly simplify the process, there are ways to work with cloud bucket data outside of the Hub, either on a local machine or remote HPC. 

### Tools

There are many tools available to interact with cloud object storage. We currently have basic operations documented for two tools:

- [fsspec](https://filesystem-spec.readthedocs.io/en/latest/) (and its submodules [gcsfs](https://gcsfs.readthedocs.io/en/latest/) and [s3fs](https://s3fs.readthedocs.io/en/latest/)) which provide filesystem-like access to local, remote, and embedded file systems from within a python session. Fsspec is also used by xarray under the hood.

- [rclone](https://rclone.org/) which provides a Command Line Interface to many different storage backends.

:::\{admonition} Note on rclone documentation
:class: tip, dropdown
Rclone is a very extensive and powerful tool, but with its many options it can be overwhelming (at least it was for Julius) at the beginning. We will only demonstrate essential options here, for more details see the [docs](https://rclone.org/). If however instructions here are not working for your specific use case, please reach out so we can improve the docs.
:::

(data.config-files)=

#### Configuration for Authenticated Access

Unless a given cloud bucket allows anonymous access or is preauthenticated within your environment (like it is the case for some of the [LEAP-Pangeo owned buckets](reference.infrastructure.buckets)) you will need to authenticate with a key/secret pair.

:::\{admonition} Always Handle credentials with care!
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

:::\{warning}
Ideally we want to store these secrets only in one central location. The natural place for these seems to be in an [AWS cli profiles](https://docs.aws.amazon.com/cli/v1/userguide/cli-configure-files.html#cli-configure-files-format), which can also be used for fsspec. There however seem to be multiple issues ([here](https://forum.rclone.org/t/shared-credentials-file-is-not-recognised/46993)) around this feature in rclone, and so far we have not succeeded in using AWS profiles in rclone.
According to those issues we can only make the aws profiles (or [source profiles?](https://forum.rclone.org/t/s3-profile-failing-when-explicit-s3-endpoint-is-present/36063/4?u=jbusecke), anyways the credentials part of it) work if we define one config file per remote [and use the 'default' profile](https://forum.rclone.org/t/shared-credentials-file-is-not-recognised/46993/2?u=jbusecke)which presumably breaks compatibility with fsspec, and also does not work at all right now. So at the moment we will have to keep the credentials in two separate spots 🤷‍♂️. **Please make sure to apply proper caution when [handling secrets](guide.code.secrets) for each config files that stores secrets in plain text!**
:::

:::\{note}
You can find more great documentation, specifically on how to use OSN resources, in [this section](https://hytest-org.github.io/hytest/essential_reading/DataSources/Data_S3.html#credentialed-access) of the [HyTEST Docs](https://hytest-org.github.io/hytest/doc/About.html#)
:::

(hub.data.setup)=
