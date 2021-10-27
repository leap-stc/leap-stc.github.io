# LEAP-Pangeo Architecture


LEAP-Pangeo is a cloud-based data and computing platform that will be used to support research, education, and knowledge transfer within the LEAP program.

## Motivation

The motivation and justification for developing LEAP-Pangeo are laid out in several recent peer-reviewed publications: {cite}`AbernatheyEtAl2021` and {cite}`GentemannEtAl2021`.
To summarize these arguments, a shared data and computing platform will:
- Empower LEAP participants with instant access to high-performance computing and analysis-ready data in order to support ambitious research objectives
- Facilitate seamless collaboration between project members around data-intensive science, accelerating research progress
- Enable rich data-driven classroom experiences for learners, helping them transition successfully from coursework to research
- Place actionable data in the hands of LEAP partners to support knowledge transfer

## Design Principles

In the proposal, we committed to building this in a way that enables the tools and infrastructure to be reused and remixed.
So The challenge for LEAP Pangeo is to deploy an “enterprise quality” platform built entirely out of open-source tools, and to make this platform as reusable and useful for the broader climate science community as possible.
We committed to following the following design principles:
- Open source
- Modular system: built out of smaller, standalone pieces which interoperate through clearly documented interfaces / standards
- Agile development on GitHub
- Following industry-standard best practices for continuous deployment, testing, etc.
- Resuse of existing technologies and contribution to "upstream" open source projects on which LEAP-Pangeo depends
  (rather than development of new stuff just for the sake of it).
  This is a key part of our sustainability plan.

## Related Tools and Platforms


It’s useful to understand the recent history and related efforts in this space.

- **[Google Colab](https://research.google.com/colaboratory/faq.html)** is a free notebook-in-the-cloud service run by Google.
  It is built around the open source Jupyter project, but with advanced notebook sharing capabilities (like Google Docs).
- **[Google Earth Engine](https://earthengine.google.org/)** is a reference point for all cloud geospatial analytics platforms.
  It’s actually a standalone application that is separate from Google Cloud, the single instance of a highly customized, black box (i.e. not open source)  application that enables parallel computing on distributed data.
  It’s very good at what it was designed for (analyzing satellite images), but isn’t easily adapted to other applications, such as machine learning.
- **[Columbia IRI Data Library](https://iridl.ldeo.columbia.edu/index.html)** is a powerful and freely accessible online data repository and analysis tool that allows a user to view, analyze, and download hundreds of terabytes of climate-related data through a standard web browser.
  Due to its somewhat outdated architecture, IRI data library cannot easily be updated or adapted to new projects.
- **[Pangeo](http://pangeo.io/)** is an open science community oriented around open-source python tools for big-data geoscience.
  It is a loose ecosystem of interoperable python packages including [Jupyter](https://jupyter.org/), [Xarray](http://xarray.pydata.org/), [Dask](http://dask.pydata.org/), and [Zarr](https://zarr.readthedocs.io/).
  The Pangeo tools have been deployed in nearly all commercial clouds (AWS, GCP, Azure) as well as HPC environments.
  [Pangeo Cloud](https://pangeo.io/cloud.html) is a publicly accessible data-proximate computing environment based on Pangeo tools.
  Pangeo is used heavily within NCAR.
- **[Microsoft Planetary Computer](https://planetarycomputer.microsoft.com/)** is a collection of datasets and computational tools hosted by Microsoft in the Azure cloud.
  It combines Pangeo-style computing environments with a data library based on [SpatioTemporal Asset Catalog](https://stacspec.org/)
- **[Radiant Earth ML Hub](https://www.radiant.earth/mlhub/)** is a cloud-based open library dedicated to Earth observation training data for use with machine learning algorithms.
  It focuses mostly on data access and curation.
  Data are cataloged using STAC.
- **[Pangeo Forge](https://pangeo-forge.org/)** is a new initiative, funded by the NSF EarthCube program, to build a platform for
  "crowdsourcing" the production of analysis-ready, cloud-optimized data.
  Once operational, Pangeo Forge will be a useful tool for many different projects which need data in the cloud.

Of these different tools, we opt to build on Pangeo because of its open-source, grassroots
foundations in the climate data science community, strong uptake within NCAR, and track-record of support from NSF.

## Design and Architecture

```{figure} https://i.imgur.com/PVhoQUu.png
---
name: architecture-diagram
---
LEAP-Pangeo high-level architecture diagram
```

There are four primary components to LEAP-Pangeo.

### The Data Library

The data library will provide analysis-ready, cloud-optimized data for all aspects of LEAP.
The data library is directly inspired by the [IRI Data Library](https://iridl.ldeo.columbia.edu) mentioned above; however, LEAP-Pangeo data will be hosted in the cloud, for maximum impact, accessibility, and interoperability.

The contents of the data library will evolve dynamically based on the needs of the project.
Examples of data that may become part of the library are
- NOAA [OISST](https://www.ncei.noaa.gov/products/optimum-interpolation-sst) sea-surface temperature data,
  used in workshops and classes to illustrate the fundamentals of geospatial data science.
- High-resolution climate model simulations from the [NCAR "EarthWorks"](https://news.ucar.edu/132760/csu-ncar-develop-high-res-global-model-community-use)
  project, used by LEAP researchers to develop machine-learning parameterizations of climate processes like cloud and ocean eddies.
- Machine-learning "challenge datasets," published by the LEAP Team and accessible to the world, to help broading participation
  by ML researchers into climate science.
- Easily accessible syntheses of climate projections from [CMIP6 data](https://esgf-node.llnl.gov/projects/cmip6/), produced by the LEAP team,
  for use by industry partners for business strategy and decision making.

### Data Storage Service

The underlying technology for the LEAP Data catalog will be cloud object storage (e.g. Amazon S3),
which enables high throughput concurrent access to many simultaneous users over the public internet.
Cloud Object Storage is the most performant, cost-effective, and simple way to serve such large volumes of data.

:::{admonition} Decision Needed
What object storage service should LEAP-Pangeo use?

The data need to be "close" to the Hub in terms of network throughput and latency.
But they don't necessarily need to live _inside_ Google Cloud.
That is both expensive to store (\$250 / TB / year) and requires egress fees (\$100 / TB)
to allow the data to _leave_ the cloud.
That could create a barrier for partners to use the data.

Alternative storage solutions include:

- [Open Storage Network](https://www.openstoragenetwork.org/) - Requires hardware purchase
- [CloudFlare R2](https://blog.cloudflare.com/introducing-r2-object-storage/) - Cheaper, no egress
:::

#### Pangeo Forge

```{figure} https://raw.githubusercontent.com/pangeo-forge/flow-charts/main/renders/architecture.png
---
width: 400px
name: pangeo-forge-flow
---
Pangeo Forge high-level workflow. Diagram from https://github.com/pangeo-forge/flow-charts
```

A central tool for the population and maintenance of the LEAP-Pangeo data catalog is
[Pangeo Forge](https://pangeo-forge.readthedocs.io/en/latest/).
Pangeo Forge is an open source tool for data Extraction, Transformation, and Loading (ETL).
The goal of Pangeo Forge is to make it easy to extract data from traditional data repositories and deposit in cloud object storage in analysis-ready, cloud-optimized (ARCO) format.

Pangeo Forge works by allowing domain scientists to define "recipes" that describe data transformation pipelines.
These recipes are stored in GitHub repositories.
Continuous integration monitors GitHub and automatically executes the data pipelines when needed.
The use of distributed, cloud-based processing allows very large volumes of data to be processed quickly.

Pangeo Forge is a new project, funded by the NSF EarthCube program.
LEAP-Pangeo will provide a high-impact use case for Pangeo Forge, and Pangeo Forge
will empower and enhance LEAP research.
This synergistic relationship with be mutually beneficial to two NSF-sponsored projects.
Using Pangeo Forge effectively will require LEAP scientists and data engineers to engage
with the open-source development process around Pangeo Forge and related technologies.

#### Catalog

A [STAC](https://stacspec.org/) data catalog be used to enumerate all LEAP-Pangeo datasets and provide this information to the public.
The catalog will store all relevant metadata about LEAP datasets following established metadata standards (e.g. CF Conventions).
It will also provide direct links to raw data in cloud object storage.

The catalog will facilitate several different modes of access:
- Searching, crawling, and opening datasets from within notebooks or scripts
- "Crawling" by search indexes or other machine-to-machine interfaces
- A pretty web front-end interface for interactive public browsing

The [Radiant Earth MLHub](https://mlhub.earth/) is a great reference for how we imagine the LEAP data catalog will eventually look.

### The Hub

```{figure} https://jupyter.org/assets/labpreview.png
---
width: 400px
name: jupyterlab-preview
---
Screenshot from JupyterLab. From <https://jupyter.org/>
```

Jupyter Notebook / Lab has emerged as the standard tool for doing interactive data science.
Jupyter supports combining rich text, code, and generated outputs (e.g. figures) into a single document, creating a way to communicate and share complete data-science research project

```{figure} https://jupyterhub.readthedocs.io/en/stable/_images/jhub-fluxogram.jpeg
---
width: 400px
name: jupyterhub-architecture
---
JupyterHub architecture. From <https://jupyterhub.readthedocs.io/>
```

JupyterHub is a multi-user Jupyter Notebook / Lab environment that runs on a server.
JupyterHub provides a gateway to highly customized software environments backed by dedicated computing with specified resources (CPU, RAM, GPU, etc.)
Running in the cloud, JupyterHub can scale up to accommodate any number of simultaneous users with no degradation in
performance.
JupyterHub environments can support basically [every existing programming language](https://github.com/jupyter/jupyter/wiki/Jupyter-kernels).
We anticipate that LEAP users will primarily use **Python**, **R**, and **Julia** programming languages.
In addition to Jupyter Notebook / Lab, JupyterHub also supports launching [R Studio](https://www.rstudio.com/).

The Pangeo project already provides [curated Docker images](https://github.com/pangeo-data/pangeo-docker-images)
with full-featured Python software environments for environmental data science.
These environments will be the starting point for LEAP environments.
They may be augmented as LEAP evolves with more specific software as needed by research projects.

:::{admonition} Decision Needed
Who should be the "identity provider" for the LEAP-Pangeo Hub?
We cannot rely on Columbia UNIs because not all partners are Columbia affiliates.
Some options:

- Google ID
- GitHub
- Globus
- ORCID
:::

### The Knowledge Graph

LEAP "outputs" will be of four main types:

- **Datasets** (covered above)
- **Papers** - traditional scientific publications
- **Project Code** - the code behind the papers, used to actually generate the scientific results
- **Trained ML Models** - models that can be used directly for inference by others
- **Educational Modules** - used for teaching

All of these object must be tracked and cataloged in a uniform way.
The {doc}`/policies/code_policy` and {doc}`/policies/data_policy` will help set these standards.

```{figure} LEAP_knowledge_graph.png
---
width: 600px
name: knowledge-graph
---
LEAP Knowledge Graph
```

By tracking the linked relationships between datasets, papers, code, models, and educational , we will generate a “knowledge graph”.
This graph will reveal the dynamic, evolving state of the outputs of LEAP research and the relationships between different elements of the project.
By also tracking participations (i.e. humans), we will build a novel and inspiring track record of LEAP's impacts through the project lifetime.

This is the most open-ended aspect of our infrastructure.
Organizing and displaying this information effectively is a challenging problem in
information architecture and systems design.
 
