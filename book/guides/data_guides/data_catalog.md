(guide.data.catalog)=

# Working with the LEAP Data Catalog

You want to have a specific dataset to explore or analyze? There is a chance that somebody else at LEAP has already worked with the data! So the first thing to look for data should always be a visit to the [LEAP Data Catalog](reference.infrastructure.catalog). This guide goes over the process of loading data from the catalog. If data is not there, it also specifies how one can request cloud-ingested data be added to the catalog. If you are completely new to data ingestion and putting data into the cloud, please check out our beginner oriented [tutorial](guides.data.ingestion_tutorial).

## How to load Data from the LEAP Catalog

This is a repository of data sets published by the LEAP community in collaboration with the Data and Compute Team. The home page will immediately show a list of which datasets are included. Every dataset has a brief description, provides a simple code snippet for loading the data into Python, and links to the original feedstock from which the data was ingested. The term ["feedstock"](https://pangeo-forge.readthedocs.io/en/latest/deployment/feedstocks.html) is inherited from the Pangeo Forge project, and basically refers to the code repository defining the data pipeline. Feedstocks allow curious users to trace back towards the original data source for transparency and reproducibility.

The basic requirements for loading the data are the following packages, which are automatically accessible to any user of the JupyterHub platform. But if you wish to load the data on your machine, then you must ensure your python environment has the following packages:

```
Xarray
gcsfs
Zarr
```

(guides.data.ingestion_pipeline)=

## How to Add a Dataset to the LEAP Catalog

To start ingesting a dataset follow these steps:

1. Let the LEAP community and the Data and Computation Team know about this new dataset. We gather all ingestion requests in our ['leap-stc/data_management' issue tracker](https://github.com/leap-stc/data-management/issues). You should check existing issues with the tag ['dataset'](https://github.com/leap-stc/data-management/issues?q=is%3Aissue+is%3Aopen+label%3Adataset) to see if somebody else might have already requested this particular dataset. If that is not the case you can add a new [dataset_request](https://github.com/leap-stc/data-management/issues/new?assignees=&labels=dataset&projects=&template=new_dataset.yaml&title=New+Dataset+%5BDataset+Name%5D). Making these request in a central location enables others to see which datasets are currently being ingested and what the status is.
1. Use our [feedstock template](https://github.com/leap-stc/LEAP_template_feedstock) to create a feedstock repository by following instructions in the README to get you started with either one of the above.
1. If issues arise please reach out to the [](support.data_compute_team)

```{note}
This does currently not provide a solution to handle datasets that have been produced by you (as e.g. part of a publication). We are working on formalizing a workflow for this type of data. Please reach out to the [](support.data_compute_team) if you have data that you would like to publish. See [](explanation.data-policy.types) for more.
```

(guide.data.upload_manual)=

### How to get new data ingested (if public download is not available)

If the source data is publicly available and accessable over https, you should create a [feedstock template](https://github.com/leap-stc/LEAP_template_feedstock). If the data is located behind a firewall on an HPC center, the 'pull' based paradigm our feedstocks will not work. In this case we have an option to 'push' the data to a special "inbox" bucket (`'leap-pangeo-inbox'`) on the OSN Pod, from there an admin can move the data to another dedicated bucket and the data can be added to the catalog using the [template feedstock](https://github.com/leap-stc/LEAP_template_feedstock).

**Step by Step instructions**

- Reach out to the [](support.data_compute_team). They will contact the OSN pod admin and share bucket credentials for the `'leap-pangeo-inbox'` bucket.
- Authenticate to that bucket from a compute location that has access to your desired data and the internet. You can find instructions on how to authenticate [here](guides.data.external.authentication.config-files).
- Upload the data to the 'leap-pangeo-inbox' in **a dedicated folder** (note the exact name of that folder, it is important for the later steps). How you exactly achieve the upload will depend on your preference. Some common options include:
  - Open a bunch of netcdf files into xarray and use `.to_zarr(...)` to write the data to zarr.
  - Use fsspec or rclone to move an existing zarr store to the target bucket
    Either way the uploaded folder should contain one or more zarr stores!
- Once you have confirmed that all data is uploaded, ask an admin to move this data to the dedicated `'leap-pangeo-manual'` bucket on the OSN pod. They can do this by running [this github action](https://github.com/leap-stc/data-management/blob/main/.github/workflows/transfer.yaml), which requires the subfolder name from above as input.
- Once the data is moved, follow the instructions in the [template feedstock](https://github.com/leap-stc/LEAP_template_feedstock) to ["link an existing dataset"](https://github.com/leap-stc/LEAP_template_feedstock#linking-existing-arco-datasets) (The actual ingestion, i.e. conversion to zarr has been done manually in this case). Reach out to the [](support.data_compute_team) if you need support.
