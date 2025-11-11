# Data Tools

There are many tools available to interact with cloud object storage. We currently have basic operations documented for three tools:

- [rclone](https://rclone.org/) which provides a Command Line Interface to many different storage backends, see [here][rclone] for more details. Rclone is highly versatile and suits almost all use cases. Details on authentication and usage can be found [here][rclone].

- GCloud SDK. One can interact with Google Cloud storage directly using the Google Cloud SDK and Command Line Interface. Please consult the [Install Instructions](https://cloud.google.com/sdk/docs/install) for more guidance.

- [fsspec](https://filesystem-spec.readthedocs.io/en/latest/) (and its submodules [gcsfs](https://gcsfs.readthedocs.io/en/latest/) and [s3fs](https://s3fs.readthedocs.io/en/latest/)) provide filesystem-like access to local, remote, and embedded file systems from within a python session. Fsspec is also used by xarray under the hood and so integrates easily with normal coding workflows (and tools like Dask).

    - `fsspec` can be used to transfer small amounts of data to google cloud storage or OSN. If you have more than a few hundred MB's, it is worth using Rclone
    - `fsspec` should be installed by default on the **Base Pangeo Notebook** environment on the JupyterHub. If you are working locally, you can install it with `pip or conda/mamba`. ex: `pip install fsspec`.

## Tool Selection

**small**: KB's to 100's of MB

**medium**: 1 to \<100GB

**large**: >100GB

#### Move data from User Directory to GCS

- Data volume:
    - small: `fsspec/gcsfs`
    - medium: `rclone`

#### Move data from JupyterHub to OSN

- If the data volume is:
    - small: `rclone`
    - medium: `rclone`

#### Move data from Laptop/HPC to OSN

- If the data volume is:
    - small: `rclone`
    - medium: `rclone`
    - large: `rclone`

## **Rclone**

Rclone is an open-source command line tool for moving and syncing data. It can be very useful for moving data into or out of LEAP cloud buckets. There is a bit of a learning-curve, but it has [extensive docs](https://rclone.org/docs/). Please [reach out to us][contact] if you run into issues.

### Usage

Rclone can be installed on the hub by typing in the Terminal:

```bash
mamba install rclone -y
```

Once installed, you can setup "remotes", which are storage locations that have credentials.
You can view and modify the rclone config file to add remotes. Rclone will show you where this is located by running: `rclone config file`. Please consult our [Authentication guide][authentication] for instructions on how to setup various remotes on rclone.

```
[leap-pubs]
type = s3
provider = Ceph
access_key_id = <CONTACT DATA AND COMPUTE TEAM>
secret_access_key = <CONTACT DATA AND COMPUTE TEAM>
endpoint = https://nyu1.osn.mghpcc.org
no_check_bucket = true

[leap-inbox]
type = s3
provider = Ceph
access_key_id = <CONTACT DATA AND COMPUTE TEAM>
secret_access_key = <CONTACT DATA AND COMPUTE TEAM>
endpoint = https://nyu1.osn.mghpcc.org
no_check_bucket = true

[leap-gcs]
type = google cloud storage
object_acl = bucketOwnerFullControl
location = us
env_auth = true
```

Details for the LEAP OSN pods are [here][open-storage-network].

You can find more great documentation, specifically on how to use OSN resources, in [this section](https://hytest-org.github.io/hytest/essential_reading/DataSources/Data_S3.html#credentialed-access) of the [HyTEST Docs](https://hytest-org.github.io/hytest/doc/About.html#)

### Listing directory contents

Rclone has file-system like `ls` syntax for cloud storage. You can list the files in a bucket/directory with:

```shell
rclone ls leap-inbox:leap-pangeo-inbox/example-dataset
```

### Moving Data

Rclone can copy data from your **local computer**, from an **HPC system**, or between **cloud buckets**.

**From Local/HPC to OSN:**

```shell
rclone copy path/to/local_or_local/dir/ leap-inbox:<leap-pangeo-inbox/dataset_name/dataset_input_files
```

**Between Cloud Buckets:**

```shell
rclone copy \
<remote_name_a>:<bucket-name>/dataset-name/dataset-files \
<remote_name_b>:<bucket-name>/dataset-name/dataset-files \
```

**From Local/HPC to GCS:**

```shell
rclone copy /path/to/local_ir_hpc/data/ \
  leap-gcs:leap-persistent/username/my_dataset
```

The rclone syntax for copying single files is slightly different then for multiple files. To copy a single file you can use the [copyto command](https://rclone.org/commands/rclone_copyto/). Another option is to copy the containing folder and use the `--include` or `--exclude` flags to select the file to copy.

!!! note

    Transfer speed is likely limited by the internet connection of your local machine.

## **fsspec/gcsfs**

[fsspec](https://filesystem-spec.readthedocs.io/en/latest/) provides a unified Python interface to local and remote filesystems.

- **gcsfs** is the backend for Google Cloud Storage (GCS).
- **s3fs** is the backend for S3-compatible systems, such as the OSN pod.

fsspec integrates directly with scientific Python libraries like **xarray**, **zarr**, and **dask**, which makes it the most seamless option for small transfers or direct read/write access from code.

### Installation

fsspec and gcsfs should already be available in the **Base Pangeo Notebook** environment on the JupyterHub.\
If working locally, install with pip or conda/mamba:

```bash
pip install fsspec gcsfs s3fs
# or
mamba install fsspec gcsfs s3fs -c conda-forge
```

### Usage

Writing to GCS:

```python
import fsspec

fs = fsspec.filesystem("gcs")  # uses gcsfs under the hood
with fs.open("gs://leap-scratch/username/test.txt", "w") as f:
    f.write("hello world")
```

Listing contents in GCS:
`fs.ls("gs://leap-scratch/username/")`

Writing to OSN (via S3 interface):

```python
import fsspec

fs = fsspec.filesystem(
    "s3",
    key="<ACCESS_KEY>",
    secret="<SECRET_KEY>",
    client_kwargs={"endpoint_url": "https://nyu1.osn.mghpcc.org"},
)
```

### Moving Data

Fsspec is great for copying smaller files from your JupyterHub home directory (/home/jovyan/...) into a LEAP GCS bucket (`gs://leap-scratch/...` or `gs://leap-persistent/...`)

Copy a single file:

```python
import fsspec
import shutil

# Local file in your JupyterHub home directory
local_path = "/home/jovyan/my_project/data.csv"

# Remote file path in GCS (your user subdirectory)
remote_path = "gs://leap-scratch/your-username/data.csv"

# Create a GCS filesystem object
fs = fsspec.filesystem("gcs")

# Open the source file and the destination file, then copy
with open(local_path, "rb") as src, fs.open(remote_path, "wb") as dst:
    shutil.copyfileobj(src, dst)
```

Copy an entire directory (e.g., project folder):

```python
import fsspec
import shutil
import os

fs = fsspec.filesystem("gcs")

local_dir = "/home/jovyan/my_project/"
remote_dir = "gs://leap-persistent/your-username/my_project/"

for root, _, files in os.walk(local_dir):
    for fname in files:
        local_file = os.path.join(root, fname)
        rel_path = os.path.relpath(local_file, local_dir)
        remote_file = remote_dir + rel_path

        with open(local_file, "rb") as src, fs.open(remote_file, "wb") as dst:
            shutil.copyfileobj(src, dst)
```

## **google-cloud-sdk**

[google-cloud-sdk](https://cloud.google.com/sdk/docs/) provides a quick to use CLI interface.

### Installation

google-cloud-sdk may already be available in the **Base Pangeo Notebook** environment on the JupyterHub.\
If not, it can be installed in the terminal as:

```bash
mamba install google-cloud-sdk
```

### Usage

The commands follow a familiar Unix-like syntax

**Writing a file to GCS:**

```bash
echo "hello world" | gcloud storage cp - gs://leap-scratch/your-username/test.txt
```

**Listing contents in GCS:**

```bash
gcloud storage ls gs://leap-scratch/your-username/
```

**Copying a single file:**

```bash
gcloud storage cp /home/jovyan/my_project/data.csv gs://leap-scratch/your-username/data.csv
```

**Copying an entire directory:**

```bash
gcloud storage cp -r /home/jovyan/my_project gs://leap-persistent/your-username/my_project
```

**Downloading from GCS:**

```bash
gcloud storage cp gs://leap-persistent/your-username/data.csv ./local_data.csv
```

**Removing files:**

```bash
gcloud storage rm gs://leap-scratch/your-username/test.txt
```

### When to Use gcloud vs fsspec

- **gcloud storage**: Best for bulk transfers, command-line workflows, and moving large directories
- **fsspec**: Best for programmatic access within Python code, direct integration with xarray/zarr/dask, and small file operations
