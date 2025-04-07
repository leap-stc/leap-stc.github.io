(guide.data.external)=

# Working outside the LEAP JupyterHub

While we primarily recommend working with LEAP data from our shared compute platform, we understand that many users have good reasons for sticking to the setups they already have. Here we describe solutions to common issues faced for interfacing with LEAP data externally, i.e. not from the JupyterHub.

## Tools for Cloud Access

There are many tools available to interact with cloud object storage. We currently have basic operations documented for two tools:

- [fsspec](https://filesystem-spec.readthedocs.io/en/latest/) (and its submodules [gcsfs](https://gcsfs.readthedocs.io/en/latest/) and [s3fs](https://s3fs.readthedocs.io/en/latest/)) which provide filesystem-like access to local, remote, and embedded file systems from within a python session. Fsspec is also used by xarray under the hood.

- [rclone](https://rclone.org/) which provides a Command Line Interface to many different storage backends.

:::\{admonition} Note on rclone documentation
:class: tip, dropdown
Rclone is a very extensive and powerful tool, but with its many options it can be overwhelming (at least it was for Julius) at the beginning. We will only demonstrate essential options here, for more details see the [docs](https://rclone.org/). If however instructions here are not working for your specific use case, please reach out so we can improve the docs.
:::

Using [`fsspec.open`](https://filesystem-spec.readthedocs.io/en/latest/api.html#fsspec.open) works similarly to the python builtin [`open`](https://docs.python.org/3/library/functions.html#open)

```python
with fsspec.open("gs://leap-scratch/funky-user/test.txt", mode="w") as f:
    f.write("hello world")
```

(data.config-files)=

## Authentication

Unless a given cloud bucket allows anonymous access or is preauthenticated within your environment, you will need to authenticate with a key/secret pair. The [LEAP-Pangeo owned buckets](reference.infrastructure.buckets) are pre-authenticated on the Hub but working with them externally requires configuation. **Note:** If you are accessing non-public cloud data from the JupyterHub (not published by LEAP), you will also have to follow this process.

:::\{admonition} Always Handle credentials with care!
:class: warning
Always handle secrets with care. Do not store them in plain text that is visible to others (e.g. in a notebook cell that is pushed to a public github repository). See [](guide.code.secrets) for more instructions on how to keep secrets safe.
:::

We recommend to store your secrets in one of the following configuration files (which will be used in the following example to read and write data):

`````{tab-set}
````{tab-item} Fsspec
Fsspec supports named [](aws profiles) in a credentials files. You can create one via Generate an aws credential file via the [aws CLI](https://docs.aws.amazon.com/cli/v1/userguide/cli-configure-files.html#cli-configure-files-examples)(installed on the hub by defaule):

```shell
aws configure --profile <pick_a_name>
```
Pick a sensible name for your profile, particularly if you are working with multiple profiles and buckets.

The file `~/.aws/credentials` then contains your key/secret similar to this:

```
[<the_profile_name_you_picked>]
aws_access_key_id = ***
aws_secret_access_key = ***
```
````

````{tab-item} Rclone
Rclone has its own [configuration file format](https://rclone.org/docs/#config-config-file) where you can specify the key and secret (and many other settings) in a similar fashion (note the missing `aws_` though!).

We recommend setting up the config file (show the default location with `rclone config file`) by hand to look something like this:

```
[<remote_name>]
... # other values
access_key_id = XXX
secret_access_key = XXX
```

You can have multiple 'remotes' in this file for different cloud buckets.

For the [](reference.infrastructrue.osn_pod) use this remote definition:

```
[osn]
type = s3
provider = Ceph
endpoint = https://nyu1.osn.mghpcc.org
access_key_id = XXX
secret_access_key = XXX
```

````
`````

:::\{warning}
Ideally we want to store these secrets only in one central location. The natural place for these seems to be in an [AWS cli profiles](https://docs.aws.amazon.com/cli/v1/userguide/cli-configure-files.html#cli-configure-files-format), which can also be used for fsspec. There however seem to be multiple issues ([here](https://forum.rclone.org/t/shared-credentials-file-is-not-recognised/46993)) around this feature in rclone, and so far we have not succeeded in using AWS profiles in rclone.
According to those issues we can only make the aws profiles (or [source profiles?](https://forum.rclone.org/t/s3-profile-failing-when-explicit-s3-endpoint-is-present/36063/4?u=jbusecke), anyways the credentials part of it) work if we define one config file per remote [and use the 'default' profile](https://forum.rclone.org/t/shared-credentials-file-is-not-recognised/46993/2?u=jbusecke)which presumably breaks compatibility with fsspec, and also does not work at all right now. So at the moment we will have to keep the credentials in two separate spots 🤷‍♂️. **Please make sure to apply proper caution when [handling secrets](guide.code.secrets) for each config files that stores secrets in plain text!**
:::

:::\{note}
You can find more great documentation, specifically on how to use OSN resources, in [this section](https://hytest-org.github.io/hytest/essential_reading/DataSources/Data_S3.html#credentialed-access) of the [HyTEST Docs](https://hytest-org.github.io/hytest/doc/About.html#)
:::

(hub.data.list)=

## Reading and Writing Data

Once authenticated, you can interface with cloud data.

`````{tab-set}
````{tab-item} Fsspec
The initial step in working with fsspec is to create a `filesystem` object which enables the abstraction on top of different object storage system.

```python
import fsspec

# for Google Storage 
fs = fsspec.filesystem('gs') # equivalent to gcsfs.GCSFileSystem()

# for s3
fs = fsspec.filesystem('s3') # equivalent to s3fs.S3FileSystem()
```

For **authenticated access** you need to pass additional arguments. In this case (for the m2lines OSN pod) we pass a custom endpoint and an [aws profile](data.config-files):
```python
fs = fsspec.filesystem(
    's3',
    profile='<the_profile_name_you_picked>',  ## This is the profile name you configured above.
    client_kwargs={'endpoint_url': 'https://nyu1.osn.mghpcc.org '} # This is the endpoint for the m2lines osn pod
)
```

You can now use the `.ls` method to list contents of a bucket and prefixes.

You can e.g. list the contents of your personal folder on the persistent GCS bucket with

```python
fs.ls("leap-persistent/funky-user") # replace with your github username
```

````

````{tab-item} Rclone

To inspect a bucket you can use clone with the profile ('remote' in rclone terminology) set up [above](data.config-files):

```shell
rclone ls <remote_name>:bucket-name/funky-user
```

````
`````

### Moving Data

`````{tab-set}
````{tab-item} Fsspec
🚧
````

````{tab-item} Rclone

You can move directories from a local computer to cloud storage with rclone (make sure you are properly [authenticated](data.config-files)):

```shell
rclone copy path/to/local/dir/ <remote_name>:<bucket-name>/funky-user/some-directory
```

You can also move data between cloud buckets using rclone

```shell
rclone copy \
 <remote_name_a>:<bucket-name>/funky-user/some-directory\
 <remote_name_b>:<bucket-name>/funky-user/some-directory
```
:::{admonition} Copying single files
:class: note
To copy single files with rclone use the [copyto command](https://rclone.org/commands/rclone_copyto/) or copy the containing folder and use the `--include` or `--exclude` flags to select the file to copy.  
:::

:::{note}
Copying with rclone will stream the data from the source to your computer and back to the target, and thus transfer speed is likely limited by the internet connection of your local machine.
:::

````
`````
