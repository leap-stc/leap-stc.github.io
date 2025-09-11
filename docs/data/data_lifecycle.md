# Data Lifecycle

This page is meant to serve as a high level reference to the data lifecycle. Please see also the overview of our primary [storage locations][where-data-lives] and their use cases. Troubleshooting of any issues will inevitably involve the specifics of your data. Questions can be addressed to the [Data and Compute Team][contact].

## Data ingestion

Working on the JupyterHub often requires transferring whatever data you want to work with into the cloud. The term "data ingestion" refers to a reproducible way to download and transform data into performant formats like Zarr. Data in LEAP's cloud storage is available for the entire LEAP community and can be added to the [LEAP Data Catalog][leaps-data-catalog].

The Data and Compute team can help with large or complicated data ingestion efforts. We organize such efforts in Github repositories called "feedstocks" that centralize knowledge for each request. Once you've verified that the data you need isn't available through the catalog:

1. Let the LEAP community and the Data and Computation Team know about the new dataset via the ['leap-stc/data_management' issue tracker](https://github.com/leap-stc/data-management/issues). You should check existing issues with the tag ['dataset'](https://github.com/leap-stc/data-management/issues?q=is%3Aissue+is%3Aopen+label%3Adataset) to see if somebody else might have already requested this particular dataset. If that is not the case you can add a new [dataset_request](https://github.com/leap-stc/data-management/issues/new?assignees=&labels=dataset&projects=&template=new_dataset.yaml&title=New+Dataset+%5BDataset+Name%5D).
1. Use our [feedstock template](https://github.com/leap-stc/LEAP_template_feedstock) to create a feedstock repository, holding the code and other information needed for ingestion, by following instructions in the README to get you started with either one of the above.

Most tasks fall into one of two categories:

### 1. Migrating Public Data

LEAP scientsts frequently want to work with publicly accessible data available in a data store like Zenodo, NASA, or NOAA. This is the use case for which much of our tooling is designed.

If the data is available via HTTP, one powerful method to avoid duplication is to create a *Virtual Zarr Store* (see [file formats][file-formats]).

Sometimes, climate data is published online but not publicly accessible, i.e. it requires some sort of credentials / authorization to access. Even if this is the case, programmatic access is often supported by tooling from the data providers. For example, the Copernicus data ecosystem has their own API and dataset licenses. User-specific access credentials or tokens stored on user directories are not accessible to other members of the hub; they remain private. However, they can be viewed by LEAP Data and Compute admins; if this is a security issue, please consult us for alternative ingestion options.

### 2. Migrating HPC Data

LEAP scientists also have access to an HPC or external filesystem on which their data is being generated. Such data might be ingested to allow collaboration with others in LEAP and/or to apply LEAP computational resources to analysis.

If the data is located behind a firewall on an HPC center, the normal 'pull' based paradigm of our feedstocks will not work. In this case we have an option to 'push' the data to a special "inbox" bucket (`'leap-pangeo-inbox'`) on the OSN Pod. Once the data is in this accessible intermediate staging area, the process and tools documented on the [feedstock template](https://github.com/leap-stc/LEAP_template_feedstock) ought to work.

The biggest barrier here is external authentication; see the [Authentication section][authentication]. Once authenticated, users can initiate a data transfer from their 'local' machine (laptop, server, or HPC Cluster) with the tools documented in [Data Tools](data-tools).

## Transferring Data (at a glance)

Use this quick guide; full details and examples are in [Data Tools](data-tools).

- **Small (KB–100s MB):** `fsspec/gcsfs` in Python (great for quick reads/writes and xarray/zarr I/O).
- **Medium/Large (GBs+):** `rclone` (robust, resumable, ideal for bucket↔bucket or local↔bucket syncing).
- **Direct GCS users:** `gsutil` via the GCloud SDK is available if you’re already in that ecosystem.

## Deleting Data

Cleaning up data is an essential part of the lifecycle. Regularly removing unneeded files helps conserve storage resources and control costs for the Hub.

!!! warning

    Depending on which cloud bucket you are working, make sure to double check which files you are deleting by inspecting the contents and only working in a subdirectory with your username (e.g. `gs://<leap-bucket>/<your-username>/some/project/structure`)

You can remove single files by using a gcsfs/fsspec filessytem as above:

```python
import gcsfs

fs = gcsfs.GCSFileSystem()  # equivalent to fsspec.fs('gs')
fs.rm("leap-persistent/funky-user/file_to_delete.nc")
```

If you want to remove zarr stores (which are an ‘exploded’ data format, and thus represented by a folder structure) you have to recursively delete the store:

`fs.rm("leap-scratch/funky-user/processed_store.zarr", recursive=True)`

### Scratch vs. Persistent vs. OSN

- `gs://leap-scratch (scratch):` Auto-deletes after 7 days. You can also remove earlier via the methods above.
- `gs://leap-persistent (persistent):` Manually delete what you no longer need to free space.
- OSN pod: Coordinate deletions with the Data and Compute Team (open a request before removing large datasets).
