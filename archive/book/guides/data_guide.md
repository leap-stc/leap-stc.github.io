(guide.data)=

# Data Guide

Data is fundamental to most people's work at LEAP. This set of guides describes best practices for how to find, read, write, transfer, ingest, and catalog data. It is meant as a broad overview to common workflows and institutional policies. We distinguish between two primary *types* of data that get worked with: "Published" and "Original" data.

- **Published Data** has been published and archived in a publicly accessible location (e.g. a data repository like [zenodo](https://zenodo.org) or [figshare](https://figshare.com)). We do not recommend uploading this data to the cloud as is, but to instead transform it to Zarr and upload it to the cloud. This ensures that the data is stored in an ARCO format and can be easily accessed by other LEAP members.

- **Original Data** is any dataset that is produced by researchers at LEAP and has not been published yet. The main use case for this data is to share it with other LEAP members and collaborate on it, as it is a *work in progress*. For original data we support direct uploaded to the cloud. *Be aware that original data could change rapidly as the data producer is iterating on their code*. We encourage all datasets to be archived and published before using them in scientific publications. The [JupyterHub Platform](reference.infrastructure.hub) is the organizing platform for Original Data.

Step one is always to check whether data already exists in the [LEAP data catalog](reference.infrastructure.catalog), in which case access is simply a matter of pasting a code snippet.

We categorize LEAP-relevant use of data into three categories:

## Onboarding to Jupyterhub

If you are a first time or new user to the LEAP platform, you may be unfamilar with how data files work in the cloud. To help onboard you to this new way of working, 2i2c has written a guide to Files and Data in the Cloud:

- [2i2c Docs: Data and Filesystem](https://docs.2i2c.org/user/topics/data/filesystem/)

We recommend you read this thoroughly, especially the part about Git and GitHub. It provides

- an overview of the different directories visible upon login to the JupyterHub and what they are meant for.
- how to authenticate to Github from the Hub and integrate it with your Hub workflow.
- an introduction to cloud object storage.

After reading this, you are well-equipped to tackle any specific task requiring data I/O and LEAP.

- Instructions on how to access data LEAP has made available are outlined in [](reference.infrastructure.catalog). This guide also overviews how to transfer data that does not exist into the catalog.
- For more generic instructions on how to publish or move your own data into cloud storage, please consult [](guides.data.ingestion).
- Once data is already placed into the Cloud, the JupyterHub platform is designed to facilitate seamless integration into data science workflows. Read more about this at [](guide.data.working)
- For whatever reasons, users might wish to navigate cloud ingested data from outside the JupyterHub or make use of non-LEAP affiliated data on our software platform. For guides on how to work externally, please consult [](guide.data.external)
