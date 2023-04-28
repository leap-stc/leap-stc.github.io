# LEAP-Pangeo JupyterHub 🚀

Our team has a cloud-based [JupyterHub](https://jupyter.org/hub).
For information who can access the hub with which privileges, please refer to
{doc}`/policies/users_roles`

|  |  |
|--|--|
| **Hub Address** | https://leap.2i2c.cloud/ |
| **Hub Location** | [Google Cloud `us-central1`](https://cloud.google.com/compute/docs/regions-zones) |
| **Hub Operator**| [2i2c](https://2i2c.org/) |
| **Hub Configuration** | https://github.com/2i2c-org/infrastructure/tree/master/config/clusters/leap |

## Getting Started

To get started using the hub, check out this video by [James Munroe](https://github.com/jmunroe) from [2i2c](https://2i2c.org) explaining the architecture.

<iframe width="560" height="315" src="https://www.youtube.com/embed/RKXWxtNqWKw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Getting Help

For questions about how to use the Hub, please use the LEAP-Pangeo discussion forum:

- https://github.com/leap-stc/leap-stc.github.io/discussions

Someone respond to your post and decide whether to refer it to 2i2c for technical support.

## Hub Usage

This is a rough and ready guide to using the Hub.
This documentation will be expanded as we learn and evolve.
Feel free to [edit it yourself](https://github.com/leap-stc/leap-stc.github.io/blob/main/book/leap-pangeo/jupyterhub.md) if you have suggetions for improvement!

### Logging In

1. 👀 Navigate to https://leap.2i2c.cloud/ and click the big orange button that says "Log in to continue"
2. 🔐 You will be prompted to authorize a GitHub application. Say "yes" to everything.
   Note you must belong to the appropriate GitHub team in order to access the hub.
   See {doc}`/policies/users_roles`  for more information.
3. 📠 You will redirect to a screen with the following options. Choose which machine type you want to use.

   <img width="410" alt="image" src="https://user-images.githubusercontent.com/1197350/167696045-d3c660c2-94d9-4a32-a8ad-61f340101ad5.png">

   You should try to use the smallest image you need.
   The GPU images should be used only when needed to accelerate model training.
4. 🕥 Wait for your server to start up. It's normal for this to take a few minutes.

### Using JupyterLab

After your server fires up, you will be dropped into a JupyterLab environment.

If you are new to JupyterLab, you might want to peruse the [user guide](https://jupyterlab.readthedocs.io/en/stable/user/interface.html).

### Shutting Down Your Server

Your server will shut down automatically after a period of inactivity.
However, if you know you are done working, it's best to shut it down directly.
To shut it down, go to https://leap.2i2c.cloud/hub/home and click the big red button that says "Stop My Server"

<img width="800" alt="image" src="https://user-images.githubusercontent.com/1197350/167768526-7742a260-d353-4bdb-b9d0-36e9cc17aba1.png">

You can also navigate to this page from JupyterLab by clicking the `File` menu and going to `Hub Control Panel`.

### The Python Environment

The Hub environment contains a full-featured, up-to-date Python environment.
The environments are maintained by Pangeo. You can read all about them at the following URL:

- https://github.com/pangeo-data/pangeo-docker-images/

These are Docker images which you can also run on your own computer.
The Hub contains a specific version of the image which can be found [here](https://github.com/2i2c-org/infrastructure/blob/master/config/clusters/leap/common.values.yaml).
For example, at the time of writing, the version of `pangeo-notebook` is `2022.05.10`.
A complete list of all packages installed in this environment is located at:

- https://github.com/pangeo-data/pangeo-docker-images/blob/2022.05.10/pangeo-notebook/packages.txt

There are separate images for pytorch and tensorflow.

You can install additional packages using `pip` and `conda`.
However, these will disappear when your server shuts down.

### Files and Data

Data and files work differently in the cloud.
To help onboard you to this new way of working, we have written a guide to Files and Data in the Cloud:

- https://docs.2i2c.org/en/latest/user/storage.html

We recommend you read this thoroughly, especially the part about Git and GitHub.

#### How can I get my data to the LEAP cloud buckets?

In order to collaboratively work on large datasets, we need to upload datasets to the cloud buckets in an ARCO (Analysis-Ready Cloud-Optimized) format like e.g. zarr (for n-dimensional arrays). 

##### Uploading data from an HPC system

A commong scenario is the following: A researcher/student has run a simulation on a High Performance Computer (HPC) at their institution, but now wants to collaboratively work on the analysis or train a machine learning model with this data. For this they need to potentially convert the data, and upload it to the cloud storage.

The following steps can be used as a guideline, but might have to be slightly modified depending on the actual setup of the users HPC

Steps executed on your <span style="color:#22B401">"local" computer (e.g. laptop)</span> will be colored in green and steps on your <span style="color:#9301B4">"remote" computer (e.g. HPC)</span> in purple.

** Conversion Script/Notebook**

In most cases you do not just want to upload the data in its current form (e.g. many netcdf files).  
<!-- TODO: Add an example of why this is bad for performance -->
Instead we will load the data into an [`xarray.Dataset`](https://docs.xarray.dev/en/stable/generated/xarray.Dataset.html) and then write that Dataset object directly to a zarr store in the cloud. 
**This is the step where your expertise on the dataset is crucial**

1. Spend some time to set up a python script/jupyter notebook that opens your files and combines them in to one or more xarray.Datasets (combine as many files as sensible into a single dataset).
Make sure that your data is lazily loaded and the `Dataset.data` is a [dask array]()
2. Make sure that the dataset looks 'right'. For example:
   - Check that the metadata is correct. Add missing information
   - Check that all the variables/dimensions are in the dataset
   - Check the dask chunksize. A general rule is to aim for around 100MB size, but the size and structure of chunking that is optimal depends heavily on the later use case. 
   <!-- Some more info on chunking? -->
3. Try to write out a subset of the data locally by calling the [`.to_zarr`](https://docs.xarray.dev/en/stable/generated/xarray.Dataset.to_zarr.html) method on the dataset. 
<!-- TODO: Warn not to write out the full dataset -->

** Upload Prerequisites**

Before we are able to set up authentication we need to make sure our HPC and local computer (required) are set up correctly.
- We manage access rights through [Google Groups](https://groups.google.com). Please contact Julius Busecke on [Slack](https://leap-nsf-stc.slack.com/team/U03MSCLCTRA) to get added to the appropriate group (a gmail address is required for this).
- Make sure to install the [Google Cloud SDK](https://cloud.google.com/sdk/docs/install) in both your HPC environment, and your local computer that can open a web browser (e.g. your laptop).

** Steps **
<span style="color:#22B401">a</span> 
<span style="color:#9301B4">b</span> in purple.
1. SSH into the HPC
2. <span style="color:#9301B4">Check that you have an internet connection with `ping www.google.com`</span>
3. <span style="color:#9301B4">Request no browser authentication: </span>
   ```
   gcloud auth application-default login --scopes=https://www.googleapis.com/auth/devstorage.read_write,https://www.googleapis.com/auth/iam.test --no-browser
   ```
4. 

### Dask

To help you scale up calculations using a cluster, the Hub is configured with Dask Gateway.
For a quick guide on how to start a Dask Cluster, consult this page from the Pangeo docs:

- https://pangeo.io/cloud.html#dask

[More info to be added soon]

### GPUs

Tier2 and Tier3 members (see [Users and Categories](../../policies/users_roles.md)) have access to a 'Large' Server instance with GPU. Currently the GPUs are [Nvidia T4](https://www.nvidia.com/en-us/data-center/tesla-t4/) models. To check what GPU is available on your server you can use [`nvidia-smi`](https://developer.nvidia.com/nvidia-system-management-interface) in the terminal window. You should get output similar to this:

```shell

   nvidia-smi
   
   +-----------------------------------------------------------------------------+
   | NVIDIA-SMI 510.47.03    Driver Version: 510.47.03    CUDA Version: 11.6     |
   |-------------------------------+----------------------+----------------------+
   | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
   | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
   |                               |                      |               MIG M. |
   |===============================+======================+======================|
   |   0  Tesla T4            Off  | 00000000:00:04.0 Off |                    0 |
   | N/A   41C    P8    11W /  70W |      0MiB / 15360MiB |      0%      Default |
   |                               |                      |                  N/A |
   +-------------------------------+----------------------+----------------------+
```
