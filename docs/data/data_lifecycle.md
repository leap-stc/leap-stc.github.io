# Data Lifecycles

If unfamiliar, you should first check out the overview of our primary [storage locations](data_locations.md) and their use cases. This page is meant to serve as a high level reference, but troubleshooting will inevitably involve the specifics of your data. If anything feels confusing or daunting your first instinct should always be reaching out to the [Data and Compute Team](../support/contact.md)!

## Getting data into the Cloud

Working on the JupyterHub generally requires transferring whatever data you want to work with into the cloud. The term "data ingestion" refers to a programmatic way to download and transform data into Analysis-Ready Cloud-Optimized (ARCO) formats. One major benefit of doing so is that the data immediately becomes available for the entire LEAP community (it is extremely easy to link ARCO data to the [LEAP Data Catalog](./data_catalog.md))!
The data ingestion process is organized around Github repositories called "feedstocks" that centralize communication with the DCT team for each request.

1. Check if the data has already been made accessible via the catalog. If not, let the LEAP community and the Data and Computation Team know about this new dataset. We gather all ingestion requests in our ['leap-stc/data_management' issue tracker](https://github.com/leap-stc/data-management/issues). You should check existing issues with the tag ['dataset'](https://github.com/leap-stc/data-management/issues?q=is%3Aissue+is%3Aopen+label%3Adataset) to see if somebody else might have already requested this particular dataset. If that is not the case you can add a new [dataset_request](https://github.com/leap-stc/data-management/issues/new?assignees=&labels=dataset&projects=&template=new_dataset.yaml&title=New+Dataset+%5BDataset+Name%5D). Making these request in a central location enables others to see which datasets are currently being ingested and what the status is.
1. Use our [feedstock template](https://github.com/leap-stc/LEAP_template_feedstock) to create a feedstock repostory by following instructions in the README to get you started with either one of the above.
1. If issues arise please reach out to the [Data and Compute Team](../support/contact.md).

Below we list the most common transfer scenarios and recommend migration workflows for each. Thinking about some of these design choices ahead of time greatly simplifies the back and forth needed by the DCT team when the request eventually comes in. But of course, we can collaborate on the best path forward given the nuances of any particular ingestion request!

### Migrating Public Data from Legacy Catalogs

Very commonly, LEAP members will want to work some publicly accessible data that might live in an existing data store like Zenodo, NASA, or NOAA. This is the classic ingestion use case for which much of our tooling is designed. If the data is available via HTTP, one powerful method to avoid duplication is to create a *Virtual Zarr Store* (see [file formats](../technical-reference/file_formats.md)).

Sometimes, climate data is published online but not publically accessible, i.e. it requires some sort of credentials / authorization to access. Even if this is the case, programmatic access is often supported by tooling from the data providers. For example, the Copernicus Data ecosystem has their own API and dataset licenses. Any user-specific access credentials or tokens stored on User Directories are not accessible to other members of the hub; they remain private. However, they can be viewed by LEAP Data and Compute admins; if this is an issue, please consult us for alternative ingestion options.

### Migrating HPC Data

Many LEAP Scientists also have access to an HPC or external filesystem on which their data is being generated. The biggest reason to ingest such data is for collaboration.
If the data is located behind a firewall on an HPC center, the normal 'pull' based paradigm of our feedstocks will not work. In this case we have an option to 'push' the data to a special "inbox" bucket (`'leap-pangeo-inbox'`) on the OSN Pod. Once the data is in this accessible intermediate staging area, the process and tools documented on the [feedstock template](https://github.com/leap-stc/LEAP_template_feedstock) ought to work.

The biggest barrier here is external authentication; see the [Authentication section](../technical-reference/authentication.md) for how to do this. Once authenticated, users can initiate a data transfer from their 'local' machine (laptop, server, or HPC Cluster) with the tools documented below.

## Data Tooling

There are many tools available to interact with cloud object storage. We currently have basic operations documented for three tools:

- GCloud SDK. One can interact with Google Cloud storage directly using the Google Cloud SDK and Command Line Interface. Please consult the [gcloud docs](https://cloud.google.com/sdk/docs/install).

- [fsspec](https://filesystem-spec.readthedocs.io/en/latest/) (and its submodules [gcsfs](https://gcsfs.readthedocs.io/en/latest/) and [s3fs](https://s3fs.readthedocs.io/en/latest/)) provide filesystem-like access to local, remote, and embedded file systems from within a python session. Fsspec is also used by xarray under the hood and so integrates easily with normal coding workflows (and tools like Dask).

  - `fsspec` can be used to transfer small amounts of data to google cloud storage or OSN. If you have more than a few hundred MB's, it is worth using Rclone
  - `fsspec` should be installed by default on the **Base Pangeo Notebook** environment on the Jupyter-Hub. If you are working locally, you can install it with `pip or conda/mamba`. ex: `pip install fsspec`.

- [rclone](https://rclone.org/) which provides a Command Line Interface to many different storage backends, see [here](../technical-reference/rclone.md) for more details.

### Tool Selection

small: KB's to 100's of MB
medium: 1 to \<100GB
large: >100GB

#### Move data from User Directory to GCS

- Data volume:
  - small: `fsspec/gcsfs`
  - medium: `rclone`

#### Move data from Jupyter-Hub to OSN

- If the data volume is:
  - small: `rclone`
  - medium: `rclone`

#### Move data from Laptop/HPC to OSN

- If the data volume is:
  - small: `rclone`
  - medium: `rclone`
  - large: `rclone`
