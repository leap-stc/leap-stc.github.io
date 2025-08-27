# File formats

This page explains some of the most common data formats used in the LEAP ecosystem.

## Zarr

Zarr is a specification for **chunked, compressed, N-dimensional array data** that can be stored in object storage (cloud buckets) or on local filesystems. Zarr enables **parallel and scalable access** to very large datasets. Unlike archival file formats such as NetCDF, the metadata is stored separately in human-readable formats like json. This enables you to access the metadata and understand the structure of the dataset without loading the data.

### Versions

There are two currently used versions, Zarr V2 and Zarr V3. In early 2025 `zarr-python`, the python library for Zarr, released a version that [supports the Zarr V3](https://zarr.dev/blog/zarr-python-3-release/) [spec](https://zarr-specs.readthedocs.io/en/latest/v3/core/). This new spec brings performance improvements and makes working with extensions much easier. The current version of `zarr-python` on `pypi` and `conda-forge` is V3. `zarr-python` V3 can still read and write V2 data.

### Example in Python:

```python
import xarray as xr

# Read in Xarray tutoral data
ds = xr.tutorial.open_dataset("air_temperature", chunks={})

# Write Zarr to GCP with Xarray
zarr_format = 3  # You can specify version 2 or version 3 here.
ds.to_zarr("gs://leap-scratch/username/air_temperature.zarr", zarr_format=zarr_format)

# Read Zarr with Xarray
roundtrip_ds = xr.open_zarr("gs://leap-scratch/username/air_temperature.zarr")
print(roundtrip_ds)
```

### Virtual Zarr Stores:

A massive amount of weather and climate data exists in "archival" file formats such as NetCDF, GRIB, HDF, TIFF etc. If these files are accessible over http, you can use the library [VirtualiZarr](https://virtualizarr.readthedocs.io/en/stable/) to create Virtual Zarr stores. This allows you to get Zarr-like access speed, **without duplicating the data**!

### Example in Python:

For detailed examples check out the [usage page](https://virtualizarr.readthedocs.io/en/stable/usage.html) on the VirtualiZarr docs.

## Icechunk

Icechunk is an open-source **transactional** version of Zarr. This allows you to version control your Zarr data, create branches, safely incrementally update your data and much more. The core Icechunk library is written in rust and has very performant I/O. You can read about it and see examples on the [Icechunk docs](https://icechunk.io/en/latest/overview/).

Think of it like "git for zarr data": you can create, version, and tag your data.

## NetCDF

Network Common Data Form (NetCDF) is one of the most established formats for storing scientific climate and weather data. It is widely used for model outputs and observational datasets because it is **self-describing, portable, and supported by most scientific tools**. While NetCDF is a super common and stable format, it is not ideal for working with in the cloud. A nice description of why can be found in this [blog](https://earthmover.io/blog/fundamentals-what-is-cloud-optimized-scientific-data).

### Example in Python:

```python
import xarray as xr

# Read in Xarray tutoral data
ds = xr.tutorial.open_dataset("air_temperature")

# Write NetCDF
ds.to_netcdf("air_temperature.nc")

# Roundtrip
ds = xr.open_dataset("air_temperature.nc")

print(ds)
```
