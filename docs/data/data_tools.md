# Data Tools

There are many tools available to interact with cloud object storage. The LEAP-Pangeo images currently support the following three tools:

- [rclone](https://rclone.org/):

    - Command-Line tool that supports many storage backends
    - Works well for **medium to very large** transfers
    - Great for copying between cloud buckets
    - Best for **GCS \<-> OSN** or **local \<-> cloud** operations
    - Details on authentication and usage can be found [here][rclone]

- google-cloud-sdk (gcloud sdk):

    - Google's official Command-Line Interface for Google Cloud Storage (GCS)
    - Very easy to use over rclone
    - Best for **bulk uploads, quick uploads, moving large directories, and command-line workflows**
    - Only works with GCS (not OSN)
    - Details on installing gcloud sdk can be found [here](https://cloud.google.com/sdk/gcloud)

- [fsspec](https://filesystem-spec.readthedocs.io/en/latest/) (and its submodules gcsfs and s3fs):

    - Provides a Python-native, filesystem-like interface to local and cloud storage
    - It's backends are used automatically by xarray, zarr, and dask, making it the most seamless way to read/write cloud directly from code
        - [gcsfs](https://gcsfs.readthedocs.io/en/latest/) --> Google Cloud Storage
        - [s3fs](https://s3fs.readthedocs.io/en/latest/) --> S3/OSN
    - Best for **small file operations, direct integration with xarray/zarr/dask, programmatic access within Python code**
    - Can install locally with `pip or conda/mamba` ex: `pip install fsspec`

## Tool Selection

**small**: KB's to 100's of MB

**medium**: 1 to \<100GB

**large**: >100GB

#### Move data from Laptop/HPC to GCS

- Data volume:
    - small: `fsspec/gcsfs` (if using Python) or `gcloud`
    - medium: `rclone` or `gcloud`
    - large: `rclone` or `gcloud`

#### Move data from JupyterHub to GCS

- Data volume:
    - small: `fsspec/gcsfs` or `gcloud`
    - medium: `rclone` or `gcloud`

#### Move data from Laptop/HPC to OSN

- If the data volume is:
    - small: `rclone` (preferred) or `fsspec/s3fs`
    - medium: `rclone`
    - large: `rclone`

#### Move data from JupyterHub to OSN

- If the data volume is:
    - small: `rclone` or `fsspec/s3fs`
    - medium: `rclone`

## Rclone

Rclone is an open-source command line tool for moving and syncing data. It can be very useful for moving data into or out of LEAP cloud buckets. There is a bit of a learning-curve, but it has [extensive docs](https://rclone.org/docs/). Please [reach out to us][contact] if you run into issues.

### Usage

Rclone is available in LEAP-Pangeo Images but can be installed locally by typing in the Terminal:

```bash
mamba install rclone -y
```

Once installed, you can setup "remotes", which are storage locations that have credentials.
You can view and modify the rclone config file to add remotes. Rclone will show you where this is located by running: `rclone config file`. Please consult our [Authentication guide][authentication] for instructions on how to setup various remotes on rclone.

### GCS Remote

```
[leap-gcs]
type = google cloud storage
object_acl = bucketOwnerFullControl
location = us
env_auth = true
```

### OSN Remotes

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

fsspec and gcsfs should already be available in the LEAP-Pangeo Images on the JupyterHub.
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

Copy an entire directory (e.g., project folder) to GCS:

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

google-cloud-sdk may already be available in the LEAP-Pangeo Images on the JupyterHub.
If not, it can be installed in the terminal as:

```bash
mamba install google-cloud-sdk
```

### Usage

The commands follow a familiar Unix-like syntax

Writing a file to GCS:

```bash
echo "hello world" | gcloud storage cp - gs://leap-scratch/your-username/test.txt
```

Listing contents in GCS:

```bash
gcloud storage ls gs://leap-scratch/your-username/
```

Copying a single file:

```bash
gcloud storage cp /home/jovyan/my_project/data.csv gs://leap-scratch/your-username/data.csv
```

Copying an entire directory:

```bash
gcloud storage cp -r /home/jovyan/my_project gs://leap-persistent/your-username/my_project
```

Downloading from GCS:

```bash
gcloud storage cp gs://leap-persistent/your-username/data.csv ./local_data.csv
```

Removing files:

```bash
gcloud storage rm gs://leap-scratch/your-username/test.txt
```
