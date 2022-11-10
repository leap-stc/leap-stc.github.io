# LEAP-Pangeo JupyterHub

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

### Dask

To help you scale up calculations using a cluster, the Hub is configured with Dask Gateway.
For a quick guide on how to start a Dask Cluster, consult this page from the Pangeo docs:

- https://pangeo.io/cloud.html#dask

[More info to be added soon]


### Common Problems (with Solutions)

#### Dask

##### "Killed Workers"

The "Killed Worker" message in dask can result due to a variety of reasons. Here are some of the common reasons why you might see such an error message

###### Datasets Chunks too large

** Issue **
The default dask worker configuration can deal well with dataset chunk sizes of ~100MB. If the chunks of your data are significantly larger, your worker might crash just upon loading a few of these chunks into memory.

[Example]()

** Solution **
You can change the configuration of your dask workers and increase the memory each worker has to deal with larger chunks. You can adjust the memory by passing additional [options] to the dask-gatway cluster upon creation:
```python
from dask_gateway import Gateway
gateway = Gateway()
options = gateway.cluster_options()

options.worker_memory = 10  # 10 GB of memory per worker.

# Create a cluster with those options
cluster = gateway.new_cluster(options)
cluster.scale(...)
client = cluster.get_client()
```
<!-- TODO: Add example how to change this in HTML repr -->

[Solved Example]()