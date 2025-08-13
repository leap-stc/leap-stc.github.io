# Authentication

Authentication is usually required only when working outside of the JupyterHub or when interfacing with the LEAP OSN pods.

## OSN Pod Authentication

**Step by Step instructions**

1. Reach out to the [the data-and-compute team][contact]. They can share bucket credentials for the `'leap-pangeo-inbox'` bucket.
1. How you exactly achieve the upload will depend on your preference. Some common options include:
    - Load / Aggregate your NetCDF files using xarray and save the output to Zarr using `.to_zarr(...)`
    - Use fsspec or rclone to move an existing zarr store to the target bucket

Whatever method you use, the final upload must contain one or more valid Zarr stores. These are required for compatibility with LEAP's cloud-optimized workflows.
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
