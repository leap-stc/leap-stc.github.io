(guide.data.working)=

# Working with Cloud Data

## JupyterHub

Once it has been ingested into a cloud bucket, working with data on the LEAP JupyterHub is incredibly easy. The most common workflow involves using [Xarray](https://docs.xarray.dev/en/stable/getting-started-guide/why-xarray.html), an open-source python project and data model for working with n-dimensional arrays. It is very useful for manipulating large gridded climate datasets. What's nice is that Xarray is built on top of well-known libraries such as NumPy and Pandas and thus can serve as a drop in "storage backend"; i.e. one can easily go Pandas \<--> Xarray \<--> Cloud.

Luckily, even if you are working from *outside* the hub, there is not that much that changes. The most difficult additional burden is authentication, which is ~automagically~ configured for LEAP resources on the Hub. To see what additional painpoints or steps are needed externally, see [](guide.data.external).

## Tools for Cloud Access

There are many tools available to interact with cloud object storage. We currently have basic operations documented for three tools:

- GCloud SDK. One can directly interact with Google Cloud storage directly using the Google Cloud SDK and Command Line Interface. If you do not have this installed, please consult the [gcloud docs](https://cloud.google.com/sdk/docs/install).

- [fsspec](https://filesystem-spec.readthedocs.io/en/latest/) (and its submodules [gcsfs](https://gcsfs.readthedocs.io/en/latest/) and [s3fs](https://s3fs.readthedocs.io/en/latest/)) which provide filesystem-like access to local, remote, and embedded file systems from within a python session. Fsspec is also used by xarray under the hood and so integrates really easily with normal coding workflows.

  - `fsspec` can be used to transfer small amounts of data to google cloud storage or OSN. If you have more than a few hundred MB's, it is worth using Rclone
  - `fsspec` should be installed by default on the **Base Pangeo Notebook** environment on the Jupyter-Hub. If you are working locally, you can install it with `pip or conda/mamba`. ex: `pip install fsspec`.

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

Using [`fsspec.open`](https://filesystem-spec.readthedocs.io/en/latest/api.html#fsspec.open) works similarly to the python builtin [`open`](https://docs.python.org/3/library/functions.html#open)

```python
with fsspec.open("gs://leap-scratch/funky-user/test.txt", mode="w") as f:
    f.write("hello world")
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

## Writing Xarray Datasets to Cloud Storage as Zarr

Below are a few snippets on how to write your Xarray Datasets to cloud storage.

````{note}
Writing to Zarr is the recommended way to save Xarray Datasets. It is very performant and scales incredibly well. That said, there are a few gotchas to look out for. Recently, [Zarr Python V3 was released](https://zarr.dev/blog/zarr-python-3-release/). Depending on which `zarr-python` version you have installed on the hub, writing can look slightly differant! As of April 2025, the pypi/conda version of Zarr python is 3+. The Python library `zarr-python` can read and write either version 2 or 3. The pangeo community image that is used for the LEAP/2i2c Jupyter-hub now comes with Zarr V3.

You can check the version with

```python
import zarr

print(zarr.__version__)
```
````

(hub.data.read_write)=

### Write to GCS (Google Cloud Storage)

We do not recommend uploading large files (e.g. netcdf) directly to the bucket. Instead we recommend to write data as ARCO (Analysis-Ready Cloud-Optimized) formats like [zarr](https://zarr.dev)(for n-dimensional arrays) and [parquet](https://parquet.apache.org)(for tabular data) (read more [here](https://ieeexplore.ieee.org/document/9354557) why we recommend ARCO formats).

Working with Xarray Datasets on the LEAP 2i2c JupyterHub makes it incredibly easy to switch storage formats when reading/writing data. You can write to a GCS bucket with Xarray with a single line of code. If you want to write to OSN, the workflow looks extremely similar. See [](guides.data.osn_pod).

```python
import xarray as xr

# Note: We are using an Xarray tutorial dataset in this example
ds = xr.tutorial.open_dataset("air_temperature", chunks={})
ds_processed = ds.mean(...).resample(...)  # sample modifications to data
# Note: This example writes to the `leap-scratch`, but you could also write to the leap-persistent bucket.
# 👀 make sure to prepend `gs://` to the path or xarray will interpret this as a local path

path = "gs://leap-scratch/<YOUR_USERNAME>/<DATASET_NAME>.zarr"

# Note: With zarr-python v3, you can write either Zarr python v2 or v3 by speciying the `zarr_format` as 2 or 3.
zarr_format = 3

# Note: Zarr python V3 currently does not support consolidated metadata.
ds_processed.to_zarr(path, zarr_format=zarr_format, consolidated=False)

# You can then open that Zarr store with Xarray with:
# roundtrip = xr.open_zarr(path, chunks={}, consolidated=False)
```

You can read it back into an xarray dataset with this snippet:

```python
import xarray as xr

ds = xr.open_dataset(
    "gs://leap-scratch/funky-user/processed_store.zarr", engine="zarr", chunks={}
)  #
```

... and you can give this to any other registered LEAP user and they can load it exactly like you can!

```{note}
Note that providing the url starting with `gs://...` assumes that you have appropriate credentials set up in your environment to read/write to that bucket. On the hub these are already set up for you to work with the [](reference.infrastructure.buckets), but if you are trying to interact with non-public buckets you need to authenticate yourself. Check out [](guides.data.external.authentication) to see an example how to do that.
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

```{warning}
Depending on which cloud bucket you are working, make sure to double check which files you are deleting by [inspecting the contents](hub.data.list) and only working in a subdirectory with your username (e.g. `gs://<leap-bucket>/<your-username>/some/project/structure`.
```

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
