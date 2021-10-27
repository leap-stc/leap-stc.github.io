# LEAP-Pangeo Implementation Plan

The different elements of the project can be implemented in parallel and gradually connected together.

## Roles

:::{admonition} Decision Needed
An open question for LEAP-Pangeo is whether to develop and maintain our infrastructure
via subcontracts or via Columbia employees whom we hire.
Below the roles are enumerated in a generic way according to the needed expertise.
:::

### Data Engineering

The {doc}`architecture`, particularly the Data Library, will require
expertise in modern data engineering, including the following areas:

- Geospatial data formats and metadata standards, including modern cloud-optimized
  formats such as Parquet and Zarr
- Geospatial data catalogs and APIs
- Cloud object storage
- Cloud automation and data pipelines
- Distributed computing frameworks for data science (e.g. Dask, Prefect, Apache Beam)
- GitHub workflows
- Continuous integration and agile development
- Track record of contribution to multi-stakeholder open-source software

Possible contractors who fit this role:

- [Development Seed](https://developmentseed.org/)
- [Element 84](https://www.element84.com/)
- [Azavea](https://www.azavea.com/)
- [Radiant Earth](https://www.radiant.earth/)

### DevOps for Cloud Hub

Developing and operating the LEAP-Pangeo JupyterHub will require the following expertise:

- Strong experience with Docker and containerization of workflows
- Deploying cloud-native applications (particularly [JupyterHub](https://zero-to-jupyterhub.readthedocs.io/en/latest/))
  using Kubernetes and Helm
- Continuous deployment using GitHub workflows
- Monitoring and optimizing cloud costs in multi-user JupyterHub environments
- Building machine-learning environments for Python and R users with tools such as
  Conda, Conda Forge, and repo2docker.
- Continuous integration and agile development
- Track record of contribution to multi-stakeholder open-source software

Possible contractors who fit this role:

- [2i2c](https://2i2c.org/)
- [Quansight](https://www.quansight.com/)

### Full-Stack Web Development

Developing the LEAP Knowledge Graph, including the library of open-source code
and machine-learning models, will require a mix of skills commonly referred to as
"full-stack web development".

- Front-end web development using HTML, CSS, and modern Javascript frameworks (e.g. React, Vue, etc.)
- Development and deployment of REST API endpoints for backend services
- Consumption of data from third-party APIs (e.g. GitHub API)
- Familiarity with Jupyter Notebook format
- Continuous integration and agile development
- Track record of contribution to multi-stakeholder open-source software

#### Education and Training

:::{admonition} Decision Needed
What is the scope of LEAP-Pangeo training? How much should we expect trainees to learn?
What is the intersection with other educational activities, including for-credit courses?
:::

### Contractors vs. Employees

## Timeline

### Fall 2021

#### Activities
- 📍 Deploy generic Pangeo JupyterHub on Google Cloud using supported credits.
- 📍 Provide basic end-user documentation for using the Hub
  (comparable to [Pangeo Cloud docs](https://pangeo.io/cloud.html)).

#### Milestones

- ✅ LEAP members log into the hub and run their first code against existing cloud data.

### Spring 2021

#### Activities

- 📍 Conduct data survey to assess data needs of research, education, and outreach activities.


#### Milestones

- ✅ LEAP researchers ingest first datasets into cloud data library.

### Summer 2021

Milestones
