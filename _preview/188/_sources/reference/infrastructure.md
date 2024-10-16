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
To accommodate the expanding LEAP community, the data and compute team has instituted a storage quota on individual user directories `/home/jovyan`. Your home directory is intended only for notebooks, analysis scripts, and small datasets (\< 1 GB). It is not an appropriate place to store large datasets. Unlike the cloud buckets, these directories use an underlying storage with a rigid limit. If a single user fills up the space, the Hub crashes for everyone. We recommend users use less than 25GB and enforce a hard limit of 50GB. **Users who persistently violate the limit may temporarily get reduced cloud access**.

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

#### Installing additonal packages

You can install additional packages using `pip` and `conda`.
However, these will disappear when your server shuts down.

For a more permanent solution we recommend building project specific dockerfiles and using those as [custom images](reference.infrastructure.hub.image.custom).

## Cloud Storage

(reference.infrastructrue.osn_pod)=

### m2lines OSN Pod

🚧

(reference.infrastructure.buckets)=

## LEAP-Pangeo Cloud Storage Buckets

LEAP-Pangeo provides users two cloud buckets to store data. Your [](reference.infrastructure.hub.server) is automatically authenticated to read from any of these buckets but write access might differ (see below). See [](reference.authentication) for details on how to access buckets from 'outside' the JupyterHub.

- `gs://leap-scratch/` - Temporary Storage deleted after 7 days. Use this bucket for testing and storing large intermediate results. [More info](https://docs.2i2c.org/user/topics/data/cloud/#scratch-bucket)
- `gs://leap-persistent/` - Persistent Storage. Use this bucket for storing results you want to share with other members.
- `gs://leap-persistent-ro/` - Persistent Storage with read-only access for most users. To upload data to this bucket you need to use [this](hub.data.upload_hpc) method below.

Files stored on each of those buckets can be accessed by any LEAP member, so be concious in the way you use these.

- **Do not put sensitive information (passwords, keys, personal data) into these buckets!**
- **When writing to buckets only ever write to your personal folder!** Your personal folder is a combination of the bucketname and your github username (e.g. \`gs://leap-persistent/funky-user/').

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
