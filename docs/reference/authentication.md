# Authentication

## Accessing LEAP Cloud Buckets

If you are working from a private server (anything that is not the Jupyterhub, i.e. local machine or HPC), accessing LEAP cloud data requires authentication. We manage bucket access directly via Google Cloud Console (GCS), on a per-user basis.

1. Ensure your private server can access the Google Cloud SDK via its Terminal. Please consult the [Install Instructions](https://cloud.google.com/sdk/docs/install) for guidance if you do not have gcloud on your machine. Login to whichever google account you want LEAP to grant access to with the [gcloud auth login](https://cloud.google.com/sdk/gcloud/reference/auth/login) command. To ensure everything worked, verify that `gcloud auth list` returns the correct account.
1. Reach out to the Data and Compute team to request access. We'll need to know which account email you have logged into gcloud with as well as [which bucket][leap-cloud-buckets] you need to access. Permissions will be granted based on what kind of access is needed (read-only, write, etc). Once confirmed on our end, verify which permissions were granted with `gcloud storage buckets get-iam-policy gs://<bucket-name>` under your account email.
1. Once you have access, you have a variety of options for actually moving the data; most tools or libraries have some way of syncing with gcloud. You may find it helpful to generate [Application Default Credentials](https://cloud.google.com/docs/authentication/application-default-credentials) using the `gcloud auth application-default login` command. This helps applications automatically find credentials at runtime, bypassing the need for manual authentication. This will create a OAuth 2.0 Token file in some location like `$HOME/gcloud/application_default_credentials.json`.

For most use cases, rclone gets the job done.

### Configuring Rclone with Google Cloud

1. Install rclone with `conda install rclone`
1. Run rclone config and follow the directives for the [google cloud storage setup](https://rclone.org/googlecloudstorage/). Generally the default options for good when in doubt, but this part can be tricky! Feel free to reach out with questions if unsure. Some guidelines:
    1. Since IAM access directly granted, you can leave the 'Project number\` blank.
    1. We do not anonymous access, that ruins the whole purpose of granting access! Put `false` when prompted.
    1. object_acl specifies the default permissions for any new objects you upload to storage. We recommend choosing either 5 or 6. bucket_acl does not matter since you're unlikely to create any new cloud buckets.
    1. If you followed the steps above and generated Application Default Credentials, you can choose `true` (**which is NOT the default**) for the "env_auth" option, which tells rclone to get GCP IAM credentials from runtime or as needed. If your `gcloud auth login` session is valid, it will be used to authenticate rclone without much hassle! Especially if you are on an HPC without a web browser, this is the best option.
1. Upon completion, you will have created or added to your rclone configuration file. To find out the location of your config file, you can run `rclone config file` (it is usually something like `$HOME/.config/rclone/rclone.conf`). If you see the remote that was just set up, you can now use rclone freely! See our [technical reference on rclone](./rclone.md) for guidance.

## OSN Pod Authentication

**Step by Step instructions**

1. Reach out to the [the data-and-compute team][contact]. They can share bucket credentials for the `'leap-pangeo-inbox'` bucket.
1. How you exactly achieve the upload will depend on your preference. Some common options include:
    - Load / Aggregate your NetCDF files using xarray and save the output to Zarr using `.to_zarr(...)`
    - Use fsspec or rclone to move an existing zarr store to the target bucket

Whatever method you use, the final upload must contain one or more valid Zarr stores. These are required for compatibility with LEAP's cloud-optimized workflows.
A typical workflow for the above steps might look like:

```python
import xarray as xr
import fsspec
import zarr

ds = xr.tutorial.open_dataset("air_temperature", chunks={})
ds_processed = ds.mean(...).resample(...)  # sample modifications to data

# define our credentials, bucket name and dataset path
osn_bucket_name = "leap-pangeo-inbox"
osn_key = "<ask DCT team>"
osn_secret = "<ask DCT team>"
endpoint_url = "https://nyu1.osn.mghpcc.org"
dataset_path = f"{osn_bucket_name}/{dataset_name}"

# Here we are using fsspec/s3fs to pass our OSN credentials to Zarr.
# Note: If you get an error like: TypeError: Unsupported type for store_like: 'FSMap'`. It is because zarr-python does not currently support the older fsspec FSMap object style. https://github.com/zarr-developers/zarr-python/issues/2706

fs = fsspec.filesystem(
    "s3",
    key=osn_key,
    secret=osn_secret,
    client_kwargs={"endpoint_url": endpoint_url},
    asynchronous=True,
)
store = zarr.storage.FsspecStore(fs, path=dataset_path)

zarr_format = 3

ds_processed.to_zarr(store=store, zarr_format=zarr_format, consolidated=False)

# Note: Your data can be read anywhere, by anyone!
# roundtrip = xr.open_zarr(f"{dataset_path}', consolidated=False)
```
