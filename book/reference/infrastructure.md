# Infrastructure

(reference.infrastructure.catalog)=

## LEAP-Pangeo Data Catalog

|                                      |                                               |
| ------------------------------------ | --------------------------------------------- |
| **Catalog Address**                  | catalog.leap.columbia.edu                     |
| **Management Repo**                  | <https://github.com/leap-stc/data-management> |
| **Maintained in collaboration with** | [Carbonplan](https://carbonplan.org)          |

For more explanation about the catalog, and its role in the overall vision of LEAP, see [](explanation.architecture). See [](guides.data.ingestion) for details on how to ingest data and link it into the catalog.

(reference.infrastructure.hub)=

## LEAP-Pangeo JupyterHub

Our team has a cloud-based [JupyterHub](https://jupyter.org/hub).
For information who can access the hub with which privileges, please refer to [](reference.membership.tiers).

|                       |                                                                                   |
| --------------------- | --------------------------------------------------------------------------------- |
| **Hub Address**       | <https://leap.2i2c.cloud/>                                                        |
| **Hub Location**      | [Google Cloud `us-central1`](https://cloud.google.com/compute/docs/regions-zones) |
| **Hub Operator**      | [2i2c](https://2i2c.org/)                                                         |
| **Hub Configuration** | <https://github.com/2i2c-org/infrastructure/tree/master/config/clusters/leap>     |

This document goes over the primary technical details of the JupyterHub.

- For a quick tutorial on basic usage, please check out our [](tutorial.getting_started) tutorial.
- To get an in-depth overview of the LEAP Pangeo Architecture and how the JupyterHub fits into it, please see the [Architecture](explanation.architecture) page.

(reference.infrastructure.hub.server)=

### Server

#### Managing Servers

You can start and stop your server (and even open multiple named servers) from the `Hub Control Panel`. You can get to the hub control panel by navigating to `https://leap.2i2c.cloud/hub/home` in your browser or navigating to `File > Hub Control Panel` from the JupyterLab Interface.

(reference.infrastructure.hub.user_dir)=

### Your User Directory

When you open your hub, you can navigate to the "File Browser" and see all the files in your User Directory
<img width="442" alt="image" src="https://github.com/leap-stc/leap-stc.github.io/assets/14314623/3ba6b45a-a077-4824-b0ec-9c111af50c33">

Your User Directory behaves very similar to a filestystem on your computer. If you save a file from a notebook, you will see it appear in the File Browser (you might have to wait a few seconds or press refresh) and you can use a terminal to navigate the terminal as you would on a UNIX machine:

<img width="357" alt="image" src="https://github.com/leap-stc/leap-stc.github.io/assets/14314623/a84c12e2-9f8a-4de1-a3e3-feff1bf59061">

:::\{note}
As shown in the picture above, every user will see `'/home/jovyan'` as their root directory. This is different from many HPC accounts where your home directory will point to a directory with your username. But the functionality is similar. These are *your own files* and they cannot be seen/modified by other users (except admins).
:::

The primary purpose of this directory is to store small files, like github repositories and other code.

:::\{warning}
To accommodate the expanding LEAP community, the data and compute team has instituted a storage quota on individual user directories `/home/jovyan`. Your home directory is intended only for notebooks, analysis scripts, and small datasets (< 1 GB). It is not an appropriate place to store large datasets. Unlike the cloud buckets, these directories use an underlying storage with a rigid limit. If a single user fills up the space, the Hub crashes for everyone. We recommend users use less than 25GB and enforce a hard limit of 50GB. **Users who persistently violate the limit may temporarily get reduced cloud access**.

To check how much space you are using in your home directory open a terminal window on the hub and run `du -h --max-depth=1 ~/ | sort -h`.

If you want to save larger files for your work use our [](reference.infrastructure.buckets) and consult our [Hub Data Guide](guide.data). See the [FAQs](faq.usr_dir_usage_warning) for guidance on reducing storage.
:::

(reference.infrastructure.hub.software_env)=

### The Software Environment

The software environment you encounter on the Hub is based upon [docker images](https://www.digitalocean.com/community/tutorials/the-docker-ecosystem-an-introduction-to-common-components) which you can run on other machines (like your laptop or an HPC cluster) for better reproducibility.

Upon start up you can choose between

- A list of preselected images
- The option of passing a custom docker image via the `"Other..."` option.

#### Preselected Images

LEAP-Pangeo uses several full-featured, up-to-date Python environments maintained by Pangeo. You can read all about them at the following URL:

- <https://github.com/pangeo-data/pangeo-docker-images/>

There are separate images for pytorch and tensorflow which are available in a drop-down panel when starting up your server.
The Hub contains a specific version of the image which can be found [here](https://github.com/2i2c-org/infrastructure/blob/master/config/clusters/leap/common.values.yaml).

For example, at the time of writing, the version of `pangeo-notebook` is `2022.05.10`.
A complete list of all packages installed in this environment is located at:

- <https://github.com/pangeo-data/pangeo-docker-images/blob/2022.05.10/pangeo-notebook/packages.txt>

:::\{attention}
We regularly update the version of the images provided in the drop-down menu.

To ensure full reproducibility you should save the full info of the image you worked with (this is stored in the environment variable `JUPYTER_IMAGE_SPEC`) with your work. You could for example print the following in the first cell of a notebook:

```python
import os

print(os.environ["JUPYTER_IMAGE_SPEC"])
```

You can then use that string with the [custom images](reference.infrastructure.hub.image.custom) to reproduce your work with exactly the same environment.
:::

(reference.infrastructure.hub.image.custom)=

#### Custom Images

If you select the `Image > Other...` Option during [server login](hub:server:login) you can paste an arbitrary reference in the form of `docker_registry/organization/image_name:image_version`. As an example we can get the `2023.05.08` version of the pangeo tensorflow notebook by pasting `quay.io/pangeo/ml-notebook:2023.05.08`.

If you want to build your own docker image for your project, take a look at [this template](https://github.com/2i2c-org/hub-user-image-template) and the instructions to learn how to use [repo2docker](https://github.com/jupyterhub/repo2docker) to set up CI workflows to automatically build docker images from your repository.

(reference.infrastructure.hub.software_env.temp_install)=

#### Installing additonal packages

You can install additional packages using `pip` and `conda`.
However, these will disappear when your server shuts down.

For a more permanent solution we recommend building project specific dockerfiles and using those as [custom images](reference.infrastructure.hub.image.custom).

(reference.infrastructure.storage)=

## Cloud Storage

We provide multiple object storage location with different characteristics and costs considerations, which are detailed below. If you plan to use any of the storage please read these instructions carefully.

:::\{warning}
The LEAP-Pangeo storage is intended to be used for [](reference.arco) data. None of the LEAP-Pangeo storage resources should be considered as a singular storage location for data.

**Never put sensitive information (passwords, keys, personal data) into any of these buckets!**
:::

(reference.infrastructrue.osn_pod)=

### m2lines OSN Pod

(reference.infrastructrue.osn_pod.organization)=

#### OSN Pod Organization

The ~1PB storage on the OSN Pod can be customized into Projects and Buckets. Projects are used to give additional users access to the Coldfront Admin Console, whereas buckets are how storage is administered up on the Pod. A project can have multiple buckets.

There are currently 3 principal Projects on the Pod:

- `'leap-pangeo'`: Used for Data Ingestion across the m2lines and LEAP community
  - Buckets:
    - `'leap-pangeo-manual'`: **No write access for users**
    - `'leap-pangeo-pipeline'`: **No write access for users**
    - `'leap-pangeo-inbox'`: *Write access can be shared with users who want to add data e.g. from an HPC center*
- `'m2lines'`: Used for project data and publications from the m2lines project
  - Buckets:
    - `'m2lines-pubs'`: **No write access for users**
    - ... various project buckets
- `'leap'`: Used for project data and publications from the LEAP project
  - Buckets:
    - `'leap-pubs'`: **No write access for users**
    - ... various project buckets

(reference.infrastructrue.osn_pod.credentials)=

#### Credentials

:::\{warning}
All OSN credentials are long lived and should be treated as such. Please do not share them publicly (e.g. in your notebook or a github repository) and when sharing with e.g. collaborators use an encrypted way of sharing (e.g. password manager).
:::

Credentials for the OSN Pod are specific to each bucket. There are two types of credentials: "Read-only" and "Read-Write". Exercise caution when sharing/saving secrets, particularly the latter. Each type of credentials consists of two keys (access + secret). Both are required to access the bucket, and they are shared by the OSN Admin.

(reference.infrastructure.buckets)=

### LEAP-Pangeo Cloud Storage Buckets

LEAP-Pangeo maintains multiple community buckets with different characteristics. Additionally we can provide research groups with access to project specific buckets.

The community buckets are currently located in Google Cloud Storage and the \[\](reference.osn-   pod).

(reference.infrastructure.buckets.google)=

#### Google Cloud Storage Buckets

The [](reference.infrastructure.hub.server) is automatically authenticated to read and write within any of the Google hosted storage. See [](reference.authentication) for details on how to access buckets from 'outside' the JupyterHub.

- `gs://leap-scratch/` - Temporary Storage deleted after 7 days. Use this bucket for testing and storing large intermediate results. [More info](https://docs.2i2c.org/user/topics/data/cloud/#scratch-bucket)
- `gs://leap-persistent/` - Persistent Storage. Use this bucket for storing results you want to share with other members.
  :::\{important}
  We will deprecate persisten storage on Google Cloud Storage in the future and be replaced by longer term scratch storage.
  :::
- `gs://leap-persistent-ro/` - Persistent Storage with read-only access for most users. This is used for our data ingested via [Pangeo-Forge](guides.data.ingestion).
  :::\{important}
  This bucket will be retired in the future. We are currently migrating the data ingestion to the [](reference.infrastructure.buckets.osn).
  :::

Files stored on each of those buckets can be accessed by any LEAP member, so be concious in the way you use these.

- **Storing data here will incur costs as long as the data exists.** Please be mindful about removing data that is not needed anymore. If you want to store large amounts of data for longer time, please [consider moving the data to one of the OSN buckets](guide.migrate_data_to_osn).
- **When writing to community buckets only ever write to your personal folder!** Your personal folder is a combination of the bucketname and your github username (e.g. \`gs://leap-persistent/funky-user/').

(reference.infrastructure.buckets.osn)=

#### OSN Buckets

The [](reference.osn-pod) is the target for our [](pangeo_forge) and [](manual) data ingestion. All of the following buckets can be read anonymously. Writing privilige is only granted to automated pipelines and to [](support.data_compute_team) members. All data in these buckets should be accessible via the [](reference.infrastructure.catalog).

- `https://nyu1.osn.mghpcc.org/leap-pangeo-pipeline` - Persistent public storage to host datasets ingested via [](guides.data.ingestion).
- `https://nyu1.osn.mghpcc.org/leap-pangeo-manual` - Persistent public storage to host datasets ingested manually. `https://nyu1.osn.mghpcc.org/leap-pangeo-inbox` is used to populate this bucket, see [](manual_ingestion) for details.

##### OSN Project Buckets

We can provide additional buckets for [](reference.m2lines) and LEAP projects. Please contact the [](support.data_compute_team) for further details.

## Compute

🚧

(reference.authentication)=

## Access to LEAP-Pangeo resources without the JupyterHub

(reference.authentication.temp_token)=

### Temporary Token

You can generate a temporary (1 hour) token with read/write access as follows:

- Now start up a [LEAP-Pangeo server](https://leap.2i2c.cloud) and open a terminal. Install the [Google Cloud SDK](https://cloud.google.com/sdk/docs/install) using mamba

```shell
mamba install google-cloud-sdk
```

Now you can generate a temporary token (valid for 1 hour) that allows you to upload data to the cloud.

```shell
gcloud auth print-access-token
```

This will print a temporary token in the terminal. You can e.g. copy that to your clipboard.

(reference.authentication.google_groups)=

### Persistent Access via Google Groups

We manage access rights through [Google Groups](https://groups.google.com). Please contact the [](support.data_compute_team) to get added to the appropriate group (a gmail address is required for this).

### Service Account

If you want more permanent access to resources, e.g. as part of a repositories CI using a service account, please reach out to the [](support.data_compute_team) to discuss options.

(reference.arco)=

## Analysis-Ready Cloud-Optimized (ARCO) Data

Below you can find some examples of ARCO data formats

### Zarr

[zarr](https://zarr.dev)

(reference.pangeo-forge)=

## Pangeo-Forge

You can find more information about Pangeo-Forge [here](https://pangeo-forge.readthedocs.io/en/latest/).

(reference.osn)=

## Open Storage Network (OSN)

🚧

(reference.m2lines)=

## m2lines

(reference.osn-pod)=

### m2lines Open Storage Network Pod

LEAP-Pangeo has access to a 1PB on-premise object storage server managed by the [](reference.osn) which is funded by the [](reference.m2lines) project. It provides the option to host data publically accessible (anonymous read) without generating [](reference.egress) costs.

## Google Cloud Storage

🚧

## General Information about Cloud Object Storage

(reference.egress)=

### Egress cost

Commodity cloud storage generally bills every single request that touches a data object. These costs are often higher when data is moved out of the providers network, commonly referred to as egress. These costs make it hard to control costs while providing data publically.
