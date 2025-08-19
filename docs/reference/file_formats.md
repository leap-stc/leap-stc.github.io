# File Formats and Data Tools

This page explains some of the most common data formats used in the LEAP ecosystem.

## netCDF

Network Common Data Form (netCDF) is one of the most established formats for storing scientific climate and weather data. It is widely used for model outputs and observational datasets because it is **self-describing, portable, and supported by most scientific tools**.

### When to Use netCDF:
- Best for local or medium-sized datasets (fits on your disk)
- Compatible with legacy tools and existing model output archives
- Good for sharing single files or smaller projects

### Example in Python:
```python
import xarray as xr

# Open a netCDF file
ds = xr.open_dataset("example_data.nc")

print(ds)
```

You can find more information on netCDF at (insert link here)

## Zarr

Zarr is a format for **chunked, compressed, N-dimensional arrays** that can be stored in object storage (cloud buckets) or local filesystems. Zarr enables **parallel and scalable access** to very large datasets without needing to download anything.

### Versions
- **Zarr v2:** The stable and widely used version today. Many datasets on cloud storage (e.g., AWS, Google Cloud) are published in this format.
- **Zarr v3:** A new standard with improved metadata handling and interoperability. Still in early adoption.

### When to Use Zarr:
- **Cloud-native workflows:** Accessing data stored on Kaggle, GCS, Pangeo or other object storage.
- **Large datasets (TB-scale):** Read only the chunks you need, not the entire dataset.
- **Parallel computing:** Works seamlessly with Dask on LEAPHub

### Virtualizarr:
Sometimes you want to *reference* a dataset without copying it. **Virtualizarr** allows you to create a *virtual Zarr store* that points to existing remote data.

Prerequisites:

- The dataset *must be accessible via HTTP(S)** (e.g., hosted on an open data portal or cloud bucket).

- Once defined, the virtual store behaves like a normal Zarr dataset and you can open it directly in Xarray.

### Example in Python:
```python
import xarray as xr

# Open a Zarr dataset hosted online
ds = xr.open_zarr("https://example-bucket/data.zarr")

print(ds)
```

You can find more information on Zarr at (insert link here)

## Icechunk Repository
Icechunk is like *Git for climate data arrays*. It provides a version control for large datasets, allowing you to store different versions efficiently without duplicating the entire dataset. 

### When to use Icechunk:
-**Remote data access:** Avoid downloading large netCDF files by referencing them directly from an HTTP or cloud-hosted location.

-**Cloud-optimized workflows:** Enable chunked and parallel reads on legacy netCDF data without transforming it into Zarr.

-**Reproducibility:** Track and version reference mappings as lightweight JSON files alongside code and notebooks.

-**Storage efficiency:** Save space by mapping existing data structures instead of duplicating them in new formats.

-**Collaboration:** Share consistent dataset references with teammates without moving or modifying source data.

### Use Case:
Suppose you have a collection of netCDF files hosted on an HTTP server. Instead of rewriting the data to Zarr, you can:

1. Use Icechunk to generate a JSON reference file that maps logical Zarr keys to byte ranges in the original netCDF files.
2. Load the virtual dataset using `xarray`:

This lets you treat your netCDFs as if they were a Zarr dataset without copying or converting them.

## Choosing the Right Tool

| Tool          | Typical Use-Cases | Pros | Cons |
|---------------|------------------|------|------|
| **netCDF**    | - Local or medium-sized datasets<br>- Legacy climate model outputs (e.g., CMIP, NOAA archives)<br>- Sharing single files | - Widely supported in scientific tools<br>- Portable and self-describing<br>- Mature ecosystem | - Not optimized for cloud/object storage<br>- Slow for very large datasets<br>- Limited parallel read/write |
| **Zarr**      | - Large, cloud-hosted datasets (e.g., ERA5 on AWS, Kaggle)<br>- Parallel analysis on LEAPHub with Dask<br>- Analysis-ready climate archives | - Cloud-native and scalable<br>- Supports chunked, selective reads<br>- Fast for distributed workflows | - Less common in legacy workflows<br>- v3 adoption still ongoing<br>- Requires object/HTTP storage |
| **Virtualizarr** | - Referencing large public datasets without duplication<br>- Creating virtual stores pointing to HTTP-accessible data | - Lightweight (metadata only)<br>- Avoids moving or copying data<br>- Behaves like a normal Zarr store in Xarray | - Requires dataset to already be HTTP-accessible<br>- Still an emerging tool |
| **Icechunk**  | - Collaborative preprocessing workflows<br>- Tracking changes to CMIP6/ERA5 datasets<br>- Creating reproducible pipelines for derived metrics | - Version control for data (like Git)<br>- Efficient storage of changes only<br>- Enables branching/merging of datasets | - Newer ecosystem with smaller community<br>- Extra setup compared to netCDF/Zarr<br>- Still experimental for some workflows |


