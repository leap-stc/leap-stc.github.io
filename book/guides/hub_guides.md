# How-To Guides for Using the Hub
These are a set of guides for using the JupyterHub Compute Environment effectively. 
## Compute
### Dask

To help you scale up calculations using a cluster, the Hub is configured with Dask Gateway.
For a quick guide on how to start a Dask Cluster, consult this page from the Pangeo docs:

- https://pangeo.io/cloud.html#dask

## Data

### I want to upload some data, where should I do it?

```{mermaid}
flowchart LR
  A[Jupyter Notebook] --> C
  B[MyST Markdown] --> C
  C(mystmd) --> D{AST}
  D <--> E[LaTeX]
  E --> F[PDF]
  D --> G[Word]
  D --> H[React]
  D --> I[HTML]
  D <--> J[JATS]
```

### I have a dataset and want to work with it on the hub. How do I upload it?

If you would like to add a new dataset to the LEAP Data Library, please first raise an issue [here](https://github.com/leap-stc/data-management/issues/new?assignees=&labels=dataset&template=new_dataset.yaml&title=New+Dataset+%5BDataset+Name%5D). This enables us to track detailed information about proposed datasets and have an open discussion about how to upload it to the cloud. 

We distinguish between two primary *types* of data to upload: "Original" and "Published" data.

- **Published Data** has been published and archived in a publically accessible location (e.g. a data repository like [zenodo](https://zenodo.org) or [figshare](https://figshare.com)). We do not recommend uploading this data to the cloud directly, but instead use [Pangeo Forge](https://pangeo-forge.readthedocs.io/en/latest/) to transform and upload it to the cloud. This ensures that the data is stored in an ARCO format and can be easily accessed by other LEAP members.
- **Original Data** is any dataset that is produced by researchers at LEAP and has not been published yet. The main use case for this data is to share it with other LEAP members and collaborate on it. For original data we support direct uploaded to the cloud. *Be aware that original data could change rapidly as the data producer is iterating on their code*. We encourage all datasets to be archived and published before using them in scientific publications.  

#### Transform and Upload published data to an ARCO format (with Pangeo Forge)

Coming Soon

(hub.guide.data.upload_manual)=
#### Upload medium sized original data from your local machine

For medium sized datasets, that can be uploaded within an hour, you can use a temporary access token generated on the JupyterHub to upload data to the cloud.

- Set up a new environment on your local machine (e.g. laptop)

```shell
mamba env create --name leap_pange_transfer python=3.9 google-auth gcsfs jupyterlab xarray zarr dask #add any other dependencies (e.g. netcdf4) that you need to read your data
```

- Activate the environment

```shell
conda activate leap_pange_transfer
```

and set up a jupyter notbook (or a pure python script) that loads your data in as few xarray datasets as possible. For instance, if you have one dataset that consists of many files split in time, you should set your notebook up to read all the files using xarray into a single dataset, and then try to write out a small part of the dataset to a zarr store.

- Now start up a [LEAP-Pangeo server](https://leap.2i2c.cloud) and open a terminal. Install the [Google Cloud SDK](https://cloud.google.com/sdk/docs/install) using mamba

```shell
mamba install google-cloud-sdk
```
Now you can generate a temporary token (valid for 1 hour) that allows you to upload data to the cloud. 

```shell
gcloud auth print-access-token
```

Copy the resulting token into a plain text file `token.txt` in a convenient location on your **local machine**.

- Now start a JupyterLab notebook and paste the following code into a cell:

```python
import gcsfs
import xarray as xr
from google.cloud import storage
from google.oauth2.credentials import Credentials

# import an access token
# - option 1: read an access token from a file
with open("path/to/your/token.txt") as f:
    access_token = f.read().strip()

# setup a storage client using credentials
credentials = Credentials(access_token)
fs = gcsfs.GCSFileSystem(token=credentials)
```

> Make sure to replace the `path/to/your/token.txt` with the actual path to your token file.

Try to write a small dataset to the cloud:

```python
ds = xr.DataArray([1]).to_dataset(name='test')
mapper = fs.get_mapper('gs://leap-scratch/<your_username>/test_offsite_upload.zarr') # This additional step is necessary to have the correct authentication set
ds.to_zarr(mapper)
```

> Replace `<your_username>` with your actual username on the hub.

- Make sure that you can read the test dataset from within the hub (go back to [Basic writing to and reading from cloud buckets](hub.data.read_write)).

- Now the last step is to paste the code to load your actual dataset into the notebook and use `.to_zarr` to upload it.

> Make sure to give the store a meaningful name, and raise an issue in the [data-management repo](https://github.com/leap-stc/data-management/issues) to get the dataset added to the LEAP Data Library.

> Make sure to use a different bucket than `leap-scratch`, since that will be deleted every 7 days! For more info refer to the available [storage buckets](hub.data.buckets).

(hub.data.upload_hpc)=
##### Uploading large original data from an HPC system (no browser access on the system available) 

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
- We manage access rights through [Google Groups](https://groups.google.com). Please contact the [](support.data_compute_team) to get added to the appropriate group (a gmail address is required for this).
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
