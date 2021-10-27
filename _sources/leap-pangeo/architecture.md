# LEAP-Pangeo Architecture


LEAP-Pangeo is a cloud-based data and computing platform that will be used to support research, education, and knowledge transfer within the LEAP program.

## Guiding Philosophy
The motivation and justification for developing LEAP-Pangeo are laid out in s recent peer-reviewed publications: [Abernathey et al. 2021](https://ieeexplore.ieee.org/abstract/document/9354557/) and [Gentemann et al. 2021](https://ieeexplore.ieee.org/abstract/document/9354557/).
To summarize these arguments, a shared data and computing platform will:
- Empower LEAP participants with instant access to high-performance computing and analysis-ready data in order to support ambitious research objectives
- Facilitate seamless collaboration between project members around data-intensive science, accelerating research progress
- Enable rich data-driven classroom experiences for learners, helping them transition successfully from coursework to research
- Place actionable data in the hands of LEAP partners to support knowledge transfer

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

- **[Google Earth Engine](https://earthengine.google.org/)** is a reference point for all cloud geospatial analytics platforms.
  It’s actually a standalone application that is separate from Google Cloud, the single instance of a highly customized, black box (i.e. not open source)  application that enables parallel computing on distributed data.
  It’s very good at what it was designed for (analyzing satellite images), but isn’t easily adapted to other applications, such as machine learning.
- **[Columbia IRI Data Library](https://iridl.ldeo.columbia.edu/index.html)** is a powerful and freely accessible online data repository and analysis tool that allows a user to view, analyze, and download hundreds of terabytes of climate-related data through a standard web browser.
  Due to its somewhat outdated architecture, IRI data library cannot easily be updated or adapted to new projects.
- **[Pangeo](http://pangeo.io/)** is an open science community oriented around open-source python tools for big-data geoscience.
  It is a loose ecosystem of interoperable python packages including [Jupyter](https://jupyter.org/), [Xarray](http://xarray.pydata.org/), [Dask](http://dask.pydata.org/), and [Zarr](https://zarr.readthedocs.io/).
  The Pangeo tools have been deployed in nearly all commercial clouds (AWS, GCP, Azure) as well as HPC environments.
  [Pangeo Cloud](https://pangeo.io/cloud.html) is a publicly accessible data-proximate computing environment based on Pangeo tools.
- **[Microsoft Planetary Computer](https://planetarycomputer.microsoft.com/)** is a collection of datasets and computational tools hosted by Microsoft in the Azure cloud.
  It combines Pangeo-style computing environments with a data library based on [SpatioTemporal Asset Catalog](https://stacspec.org/)
- **[Radiant Earth ML Hub](https://www.radiant.earth/mlhub/)** is a cloud-based open library dedicated to Earth observation training data for use with machine learning algorithms.
  It focuses mostly on data access and curation.
  Data are cataloged using STAC.
- **[Pangeo Forge](https://pangeo-forge.org/)** is a new initiative, funded by the NSF EarthCube program, to build a platform for
  "crowdsourcing" the production of analysis-ready, cloud-optimized data.
  Once operational, Pangeo Forge will be a useful tool for many different projects which need data in the cloud.



## Design and Architecture

![Architecture](https://i.imgur.com/PVhoQUu.png)

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

Data will be ingested into the library using [Pangeo Forge](https://pangeo-forge.readthedocs.io/en/latest/) and converted to cloud-optimized formats.
A [STAC](https://stacspec.org/) data catalog will allow web-based browsing as well as API-based search and access.
The LEAP-Pangeo catalog will integrate with the broader Pangeo Forge data ecosystem.

### The JupyterHub

Jupyter Notebook / Lab has emerged as the standard tool for doing interactive data science.
Jupyter supports combining rich text, code, and generated outputs (e.g. figures) into a single document, creating a way to communicate and share complete data-science research project
JupyterHub is a multi-user Jupyter notebook / lab environment that runs on a server.
Running in the cloud, JupyterHub provides a gateway to highly customized software environments backed by dedicated computing with specified resources (CPU, RAM, GPU, etc.)

![JupyterLab Preview](https://jupyter.org/assets/labpreview.png)

Our JupyterHub will be the virtual "water cooler" of LEAP.
By providing all participants with a common computing environment, we will facilitate seamless collaboration between team members.


Sub-components of the JupyterHub include
- Environment library (shared environmen specification / Docker images)
- BinderHub, to allow others to reproduce the LEAP research in the cloud


### The Notebook / Code / Model Library
All code for LEAP projects will be shared and archived in a uniform way. Researchers will be able to easily share their code with colleagues and then publish it publicly.
The Publication Database
We will try to make all our publications open access. We will keep a database of every publication, with links to the code and data that went into it.

### The Knowledge Graph
By tracking the linked relationships between papers, code, and data, we will generate a “knowledge graph”.
This graph will reveal the dynamic, evolving state of the outputs of LEAP research and the relationships between different elements of the project.
