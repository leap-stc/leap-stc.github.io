(guide.data.working)=

# Working with Cloud Data on the JupyterHub

Once it has been ingested into a cloud bucket, working with data on the LEAP JupyterHub is incredibly easy. The most common workflow involves using [Xarray](https://docs.xarray.dev/en/stable/getting-started-guide/why-xarray.html), an open-source python project and data model for working with n-dimensional arrays. It is very useful for manipulating large gridded climate datasets. What's nice is that Xarray is built on top of well-known libraries such as NumPy and Pandas and thus can serve as a drop in "storage backend"; i.e. one can easily go Pandas \<--> Xarray \<--> Cloud.

## Using the LEAP Data Catalog

You want to have a specific dataset to explore or analyze? There is a chance that somebody else at LEAP has already worked with the data! So the first thing to look for data should always be a visit to the [LEAP Data Catalog](reference.infrastructure.catalog). If you are completely new to data ingestion and putting data into the cloud, please check out our beginner oriented [tutorial](tutorials.data.ingestion) or our [guide on transferring Cloud Data](guides.data.ingestion).

### Loading a Dataset

This is a repository of data sets published by the LEAP community in collaboration with the Data and Compute Team. The home page will immediately show a list of which datasets are included. Every dataset has a brief description, provides a simple code snippet for loading the data into Python, and links to the original feedstock from which the data was ingested. The term ["feedstock"](https://pangeo-forge.readthedocs.io/en/latest/deployment/feedstocks.html) is inherited from the Pangeo Forge project, and basically refers to the code repository defining the data pipeline. Feedstocks allow curious users to trace back towards the original data source for transparency and reproducibility.

The basic requirements for loading the data are the following packages, which are automatically accessible to any user of the JupyterHub platform. But if you wish to load the data on your machine, then you must ensure your python environment has the following packages:

```
Xarray
gcsfs
Zarr
```

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

Working with Xarray Datasets on the LEAP 2i2c JupyterHub makes it incredibly easy to switch storage formats when reading/writing data. You can write to a GCS bucket with Xarray with a single line of code as the authentication should be ~automagically~ configured.

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

#### Write to OSN - Open Storage Network

LEAP has an allocation of storage on an OSN pod. OSN allows s3-like cloud storage that has no egress fees. This means that you can share data with the public or outside colaborators without any cost per request. Please contact the data-and-compute team on slack if you feel like this would fit your data use case and you want to store data on OSN.

In the example below, we have to provide a bit more authentication to write to OSN.

```{admonition} Note on OSN
---
class: tip, dropdown
---
This section is a work in progress and may change. Generally leap users should default to writing to GCS.
```

```python
import xarray as xr
import fsspec
import zarr

ds = xr.tutorial.open_dataset("air_temperature", chunks={})

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

ds.to_zarr(store=store, zarr_format=zarr_format, consolidated=False)

# Note: Your data can be read anywhere, by anyone!
# roundtrip = xr.open_zarr(f"{dataset_path}', consolidated=False)
```

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
