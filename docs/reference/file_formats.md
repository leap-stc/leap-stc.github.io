# File Formats

This page explains some of the most common data formats used in the LEAP ecosystem.

## netCDF

Network Common Data Form (netCDF) is one of the most established formats for storing scientific climate and weather data. It is widely used for model outputs and observational datasets because it is **self-describing, portable, and supported by most scientific tools**.

### When to Use netCDF:

- Best for local or medium-sized datasets (fits on your disk)
- Ideal for working with climate model outputs or archival formats (e.g., CMIP, NOAA)
- Convenient for sharing as single downloadable files or for smaller-scale workflows

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

- **Zarr v2:** The stable and widely used version today. Many datasets on cloud storage (e.g., AWS, Google Cloud) are published in this format. We recommend using this version since it is compatible with LEAP Hub.
- **Zarr v3:** A new standard with improved metadata handling and interoperability. Still in early adoption.

### When to Use Zarr:

- Accessing large data stored on Kaggle, GCS, Pangeo or other object storage.
- Enables selective chunked access without full dataset downloads
- Supports scalable, parallel computation environments like Dask on LEAPHub

### Virtual Zarr Stores:

You can create a "virtual Zarr store that points to existing remote files without copying them. This is useful when referencing datasets that are already hosted on public cloud platforms (via HTTP/S)

Prerequisites:

- The dataset *must be accessible via HTTP(S)* (e.g., hosted on an open data portal or cloud bucket).

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

Icechunk is a tool that lets you **map existing netCDF files into a virtual Zarr structure**, using a JSON reference file. It enables you to treat legacy formats like netCDF as if they were Zarr without copying or transforming them.

Think of it like "Git for data": you can create, version, and share mappings to large datasets with minimal overhead.

### When to use Icechunk:

- Avoiding downloading large netCDF files by referencing them directly from an HTTP or cloud-hosted location.

- Enabling chunked and parallel reads on netCDF data without transforming it into Zarr.

- Supports collaborating by sharing lightweight references instead of large datasets

- Reduces storage costs by avoiding data duplication across teams or pipelines

### Use Case:

Suppose you have a collection of netCDF files hosted on an HTTP server. Instead of downloading or converting them into Zarr, you can create a lightweight virtual mapping that behaves like a Zarr dataset.

Example Workflow:

- Use Icechunk to generate a JSON reference file (`reference.json`) that maps logical Zarr keys to byte ranges in the original netCDF files.
- Load the virtual dataset as if it were a Zarr store using `xarray` and `fsspec`:

```python
import xarray as xr
import fsspec

fs = fsspec.filesystem("reference", fo="reference.json")
mapper = fs.get_mapper("")
ds = xr.open_zarr(mapper, consolidated=False)

print(ds)
```

This approach enables fast, parallel access to remote netCDF files without transforming or duplicating the original data.

You can find more information on Icechunk [here](https://github.com/earth-mover/icechunk)

## Choosing the Right Tool

| Tool             | Typical Use-Cases                                                                                                                             | Pros                                                                                                                    | Cons                                                                                                                         |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **netCDF**       | - Local or medium-sized datasets<br>- Legacy climate model outputs (e.g., CMIP, NOAA archives)<br>- Sharing single files                      | - Widely supported in scientific tools<br>- Portable and self-describing<br>- Mature ecosystem                          | - Not optimized for cloud/object storage<br>- Slow for very large datasets<br>- Limited parallel read/write                  |
| **Zarr**         | - Large, cloud-hosted datasets (e.g., ERA5 on AWS, Kaggle)<br>- Parallel analysis on LEAPHub with Dask<br>- Analysis-ready climate archives   | - Cloud-native and scalable<br>- Supports chunked, selective reads<br>- Fast for distributed workflows                  | - Less common in legacy workflows<br>- v3 adoption still ongoing<br>- Requires object/HTTP storage                           |
| **Virtualizarr** | - Referencing large public datasets without duplication<br>- Creating virtual stores pointing to HTTP-accessible data                         | - Lightweight (metadata only)<br>- Avoids moving or copying data<br>- Behaves like a normal Zarr store in Xarray        | - Requires dataset to already be HTTP-accessible<br>- Still an emerging tool                                                 |
| **Icechunk**     | - Collaborative preprocessing workflows<br>- Tracking changes to CMIP6/ERA5 datasets<br>- Creating reproducible pipelines for derived metrics | - Version control for data (like Git)<br>- Efficient storage of changes only<br>- Enables branching/merging of datasets | - Newer ecosystem with smaller community<br>- Extra setup compared to netCDF/Zarr<br>- Still experimental for some workflows |
