(guide.data)=

# Data Guide

Data is fundamental to most people's work at LEAP. This set of guides describes best practices for how to find, read, write, transfer, ingest, and catalog data. It is meant as a broad overview to common workflows and institutional policies. We categorize LEAP-relevant use of data into three categories:

- Publishing scientific data for consumption by other parties. This includes ingesting data into [public cloud buckets](reference.infrastructure.buckets) from existing data stores that may be hard to access/work with.
- Working with data that already exists in the cloud. This is in conjunction with the [JupyterHub platform](reference.infrastructure.hub).
- Using the data that LEAP has catalogued and made accessible in ARCO format. This is done via our [Data Catalog](reference.infrastructure.catalog).

## Onboarding to Jupyterhub

If you are a first time or new user to the LEAP platform, you may be unfamilar with how data files work in the cloud. To help onboard you to this new way of working, 2i2c has written a guide to Files and Data in the Cloud:

- [2i2c Docs: Data and Filesystem](https://docs.2i2c.org/user/topics/data/filesystem/)

We recommend you read this thoroughly, especially the part about Git and GitHub. It provides

- an overview of the different directories visible upon login to the JupyterHub and what they are meant for.
- how to authenticate to Github from the Hub and integrate it with your Hub workflow.
- an introduction to cloud object storage.

After reading this, you are well-equipped to tackle any specific task requiring data I/O and LEAP.

- Instructions on how to access data LEAP has made available are outlined in [](guide.data.catalog). This guide also overviews how to transfer data that does not exist into the catalog.
- For more generic instructions on how to publish or move your own data into cloud storage, please consult [](guide.data.transfer).
- Once data is already placed into the Cloud, the [JupyterHub platform](explanation.jupyterhub) is designed to facilitate seamless integration into data science workflows. Read more about this at [](guide.data.working)
- For whatever reasons, users might wish to navigate cloud ingested data from outside the JupyterHub or make use of non-LEAP affiliated data on our software platform. For guides on how to access generic buckets externally, please consult [](guide.data.external)
