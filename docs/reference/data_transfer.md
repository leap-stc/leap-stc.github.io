# Data Transfer

## Rclone

Rclone is an open-source command line tool for moving and syncing data. It can be very useful for moving data into or out of LEAP cloud buckets. There is a bit of a learning-curve, but it has [extensive docs](https://rclone.org/docs/). Please [reach out to us][contact] if you run into issues.

## Usage

Rclone can be installed on the hub by typing in the Terminal:

```bash
mamba install rclone -y
```

Once installed, you can setup "remotes", which are storage locations that have credentials.
You can view and modify the rclone config file to add remotes. Rclone will show you where this is located by running: `rclone config file`. Please consult our [Authentication guide](./authentication.md) for instructions on how to setup various remotes on rclone.

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

You can move directories from a local computer to cloud storage with rclone.

```shell
rclone copy path/to/local/dir/ leap-inbox:<leap-pangeo-inbox/dataset_name/dataset_input_files
```

You can also move data between cloud buckets using rclone

```shell
rclone copy \
<remote_name_a>:<bucket-name>/dataset-name/dataset-files \
<remote_name_b>:<bucket-name>/dataset-name/dataset-files \
```

The rclone syntax for copying single files is slightly different then for multiple files. To copy a single file you can use the [copyto command](https://rclone.org/commands/rclone_copyto/). Another option is to copy the containing folder and use the `--include` or `--exclude` flags to select the file to copy.

!!! note

    Transfer speed is likely limited by the internet connection of your local machine.
