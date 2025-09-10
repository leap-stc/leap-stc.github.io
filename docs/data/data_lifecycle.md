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

The biggest barrier here is external authentication; see the [Authentication section][authentication]. Once authenticated, users can initiate a data transfer from their 'local' machine (laptop, server, or HPC Cluster) with the tools documented below.

## Data Tooling

There are many tools available to interact with cloud object storage. We currently have basic operations documented for three tools:

- [rclone](https://rclone.org/) which provides a Command Line Interface to many different storage backends, see [here](../reference/data_transfer.md) for more details. Rclone is highly versatile and suits almost all use cases. Details on authentication and usage can be found [here][rclone].

- GCloud SDK. One can interact with Google Cloud storage directly using the Google Cloud SDK and Command Line Interface. Please consult the [Install Instructions](https://cloud.google.com/sdk/docs/install) for more guidance.

- [fsspec](https://filesystem-spec.readthedocs.io/en/latest/) (and its submodules [gcsfs](https://gcsfs.readthedocs.io/en/latest/) and [s3fs](https://s3fs.readthedocs.io/en/latest/)) provide filesystem-like access to local, remote, and embedded file systems from within a python session. Fsspec is also used by xarray under the hood and so integrates easily with normal coding workflows (and tools like Dask).

    - `fsspec` can be used to transfer small amounts of data to google cloud storage or OSN. If you have more than a few hundred MB's, it is worth using Rclone
    - `fsspec` should be installed by default on the **Base Pangeo Notebook** environment on the JupyterHub. If you are working locally, you can install it with `pip or conda/mamba`. ex: `pip install fsspec`.

### Tool Selection

**small**: KB's to 100's of MB

**medium**: 1 to \<100GB

**large**: >100GB

#### Move data from User Directory to GCS

- Data volume:
    - small: `fsspec/gcsfs`
    - medium: `rclone`

#### Move data from JupyterHub to OSN

- If the data volume is:
    - small: `rclone`
    - medium: `rclone`

#### Move data from Laptop/HPC to OSN

- If the data volume is:
    - small: `rclone`
    - medium: `rclone`
    - large: `rclone`

(Insert links to Rclone and fsspec guides)
