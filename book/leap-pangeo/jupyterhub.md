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

### Office Hours

We also offer in-person and virtual Office Hours on Thursdays for questions about LEAP-Pangeo.
You can reserve an appointment [here](https://app.reclaim.ai/m/leap-pangeo-office-hours).

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

- [2i2c Docs: Data and Filesystem](https://docs.2i2c.org/data/index.html#data-and-filesystem)

We recommend you read this thoroughly, especially the part about Git and GitHub.

:::{warning}
Please do not store large files in your user directory `/home/jovyan`. Your home directory is intended only for notebooks, analysis scripts, and small datasets (< 1 GB). It is not an appropriate place to store large datasets.
:::

#### LEAP-Pangeo Buckets

LEAP-Pangeo provides users two cloud buckets to store data

- `gs://leap-scratch/` - Temporary Storage deleted after 7 days. Use this bucket for testing and storing large intermediate results. [More info](https://docs.2i2c.org/data/cloud/#scratch-bucket)
- `gs://leap-persistent` - Persistent Storage. Use this bucket for storing results you want to share with other members. 

Files stored on each of those buckets can be accessed by any LEAP member, so be concious in the way you use these.

- **Do not put sensitive information (passwords, keys, personal data) into these buckets!**
- **When writing to buckets only ever write to your personal folder!** Your personal folder is a combination of the bucketname and your github username (e.g. `gs://leap-persistent/some-really-funky-user/').

#### Inspecting contents of the bucket

We recommend using [gcsfs](https://gcsfs.readthedocs.io/en/latest/) or [fsspec](https://filesystem-spec.readthedocs.io/en/latest/) which provide a filesytem-like interface for python.

You can e.g. list the contents of your personal folder with 
```python
import gcsfs
fs = gcsfs.GCSFileSystem() # equivalent to fsspec.fs('gs')
fs.ls('leap-persistent/octocat')
```

#### Basic writing to and reading from cloud buckets

We do not recommend uploading large files (e.g. netcdf) directly to the bucket. Instead we recommend to write data as ARCO (Analysis-Ready Cloud-Optimized) formats like [zarr](https://zarr.dev)(for n-dimensional arrays) and [parquet](https://parquet.apache.org)(for tabular data) (read more [here](https://ieeexplore.ieee.org/document/9354557) why we recommend ARCO formats).

If you work with xarray Datasets switching the storage format is as easy as swapping out a single line when reading/writing data:

Xarray provides a method to stream results of a computation to zarr
```python
ds = ...
ds_processed = ds.mean(...).resample(...)
user_path = "gs://leap-scratch/some-really-funky-user" # 👀 make sure to prepend `gs://` to the path or xarray will interpret this as a local path
store_name = "processed_store.zarr"
ds_processed.to_zarr(f'{user_path}/{store_name}')
```
This will write a zarr store to the scratch bucket. 

You can read it back into an xarray dataset with this snippet:
```python
import xarray as xr
ds = xr.open_dataset('gs://leap-scratch/some-really-funky-user/processed_store.zarr', engine='zarr', chunks={}) #
```
... and you can give this to any other registered LEAP user and they can load it exactly like you can!  


#### I have a dataset and want to work with it on the hub. How do I upload it?

If you would like to add a new dataset to the LEAP Data Library, please first raise an issue [here](https://github.com/leap-stc/data-management/issues/new?assignees=&labels=dataset&template=new_dataset.yaml&title=New+Dataset+%5BDataset+Name%5D). This enables us to track detailed information about proposed datasets and have an open discussion about how to upload it to the cloud. 

Below you can find instructions for different use cases:

##### Transform and Upload archived data to an ARCO format (with Pangeo Forge)

Coming Soon


##### Uploading data from an HPC system 

A commong scenario is the following: A researcher/student has run a simulation on a High Performance Computer (HPC) at their institution, but now wants to collaboratively work on the analysis or train a machine learning model with this data. For this they need to upload it to the cloud storage.

The following steps will guide you through the steps needed to authenticate and upload data to the cloud, but might have to be slightly modified depending on the actual setup of the users HPC.

**Conversion Script/Notebook**

In most cases you do not just want to upload the data in its current form (e.g. many netcdf files).  
<!-- TODO: Add an example of why this is bad for performance -->
Instead we will load the data into an [`xarray.Dataset`](https://docs.xarray.dev/en/stable/generated/xarray.Dataset.html) and then write that Dataset object directly to a zarr store in the cloud. For this you need a python environment with `xarray, gcsfs, zarr` installed (you might need additional dependencies for your particular use case).

1. Spend some time to set up a python script/jupyter notebook on the HPC system that opens your files and combines them in to one or more xarray.Datasets (combine as many files as sensible into a single dataset). Make sure that your data is lazily loaded and the `Dataset.data` is a [dask array](https://docs.dask.org/en/stable/array.html)

2. Check your dataset:
   - Check that the metadata is correct.
   - Check that all the variables/dimensions are in the dataset
   - Check the dask chunksize. A general rule is to aim for around 100MB size, but the size and structure of chunking that is optimal depends heavily on the later use case. 
   <!-- Some more info on chunking? -->

3. Try to write out a subset of the data locally by calling the [`.to_zarr`](https://docs.xarray.dev/en/stable/generated/xarray.Dataset.to_zarr.html) method on the dataset.
<!-- TODO: Warn not to write out the full dataset -->
Once that works we can move on to the authentication.

**Upload Prerequisites**

Before we are able to set up authentication we need to make sure our HPC and local computer (required) are set up correctly.
- We manage access rights through [Google Groups](https://groups.google.com). Please contact Julius Busecke on [Slack](https://leap-nsf-stc.slack.com/team/U03MSCLCTRA) to get added to the appropriate group (a gmail address is required for this).
- Make sure to install the [Google Cloud SDK](https://cloud.google.com/sdk/docs/install) in both your <span style="color:#9301B4">HPC environment</span>, and your local computer that can open a web browser (e.g. your laptop).

**Steps**
Steps executed on your <span style="color:#22B401">"local" computer (e.g. laptop)</span> will be colored in green and steps on your <span style="color:#9301B4">"remote" computer (e.g. HPC)</span> in purple.

1. SSH into the HPC
2. <span style="color:#9301B4">Check that you have an internet connection with `ping www.google.com`</span>
3. <span style="color:#9301B4">Request no browser authentication: </span>
   ```
   gcloud auth application-default login --scopes=https://www.googleapis.com/auth/devstorage.read_write,https://www.googleapis.com/auth/iam.test --no-browser
   ```
   > 🚨 It is very important to include the `--scopes=` argument for security reasons. Do not run this command without it!
4. Follow the onscreen prompt and paste the command into a terminal on your local machine.
5. <span style="color:#22B401">This will open a browser window. Authenticate with the gmail account that was added to the google group. </span>
6. <span style="color:#22B401">Go back to the terminal and follow the onscreen instructions.</span> Copy the text from the command line and <span style="color:#9301B4">paste the command in the open dialog on the remote machine.</span>
7. <span style="color:#9301B4">Make sure to note the path to the auth json!</span> It will be something like `.../.config/gcloud/....json`.</span>

Now you are have everything you need to authenticate.

Lets verify that you can write a small dummy dataset to the cloud. In your notebook/script run the following (make sure to replace the filename and your username as instructed).

Your dataset should now be available for all LEAP members 🎉🚀

```python
import xarray as xr
import gcsfs
import json

with open("your_auth_file.json") as f: #🚨 make sure to enter the `.json` file from step 7
	token=json.load(f)

# test write a small dummy xarray dataset to zarr
ds = xr.DataArray([1, 4, 6]).to_dataset(name='data') 
# Once you have confirmed 

fs = gcsfs.GCSFileSystem(token=token)
mapper = fs.get_mapper("gs://leap-persistent/<username>/testing/demo_write_from_remote.zarr") #🚨 enter your leap (github) username here
ds.to_zarr(mapper)
```

Now you can repeat the same steps but replace your dataset with the full dataset from above and leave your python code running until the upload has finished. Depending on the internet connection speed and the size of the full dataset, this can take a while. 

If you want to see a progress bar, you can wrap the call to `.to_zarr` with a [dask progress bar](https://docs.dask.org/en/stable/diagnostics-local.html#progress-bar)

```python
from dask.diagnostics import ProgressBar
with ProgressBar():
   ds.to_zarr(mapper)
```

Once the data has been uploaded, make sure to erase the `.../.config/gcloud/....json` file from step 7, and ask to be removed from the Google Group. 

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
