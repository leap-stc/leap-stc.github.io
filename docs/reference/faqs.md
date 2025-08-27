# FAQs

## Where is my Invitation?

Please check your email account (**the one you used to sign up for Github** - this is independent of the email you use for LEAP) for an invite that will look similar to this:

![Email Invite](../assets/email_org_invite.png)

Click the link and accept all invites.

Alternatively you can log into your github account, and should see a notification in the top right menu under the "Organizations" tab.

![Github org invite 1](../assets/gh_org_invite_1.png)

You can follow that and accept the invitation there aswell.

![Github Org invite 2](../assets/gh_org_invite_2.png)

## I cannot log into the hub 😱

If you are unable to log into the hub, please check the following steps:

- [ ] Check if you are member of the [appropriate github teams]\<MISSING SECTION?>.

If you **are not** follow these steps:

- [ ] Did you [sign up for LEAP membership]\<MISSING SECTION?> This will be done for you if you sign up for an event like the Momentum bootcamp!
- [ ] Did you receive a github invite? See above for how to check for that.
- [ ] Check again if they are part of the appropriate github teams.
- If these steps do not work, please reach out to the [Data and Compute Team][contact].

If you **are** member of one of the github teams, ask them to try the following steps:

- [ ] Refresh the browser cache
- [ ] Try a different browser
- [ ] Restart the computer
- If these steps do not work, please reach out to the [Data and Compute Team][contact].

## I received a warning about space on my User Directory

If you get a Hub Usage Alert email, this means you are violating the User Directory storage limit (to learn why this limit exists, see read about [User Directories][your-jupyterhub-user-directory]). Remember that user directories are for scripts and notebooks not datasets! Users who persistently violate hub usage policies may temporarily get reduced cloud access.

**Troubleshooting**

- To see which files and directories are taking up the bulk of your storage, run `du -h --max-depth=1 ~/ | sort -h` in Terminal. It will likely reveal cached files and small/medium size data files that can be removed without disrupting typical usage.
- Delete cached files, ipython checkpoints, and any other unwanted files.
- If you still require more storage, it is likely that you are storing downloaded data in your user directory. We recommend storing data in a LEAP cloud bucket or data catalog. For more information, please consult our [data locations][where-data-lives].

Our goal is to accommodate all community members and thus we are happy to assist users in relocating data. If you have any concerns, please reach out to the [Data and Compute Team][contact].

## Dask / Xarray tips and tricks

The Xarray docs have a page on [Xarray + dask best practices](https://docs.xarray.dev/en/stable/user-guide/dask.html#best-practices).

#### Correct chunk sizes

When working with Xarray and Zarr, you should aim for data chunk sizes around **100MB**. Chunk sizes too small can overwhelm the Dask scheduler, while chunk sizes being to large can give you memory issues.

You can examine the chunk size and shape by viewing the html repr of an Xarray dataset in a Jupyter Notebook. If you click on the right-most database logo you should get a drop-down menu that shows the chunking information.

![Chunking](../assets/xarray_repr.png)

#### Delay computation until write

Xarray is great at lazy-computation, this means that is usually possible to run multiple operations before any computation is done. Generally it is a good idea to keep everything lazy (not eager!) until your finish your processing and write your data. For example, a `to_zarr()` call would trigger the computation. This can generally be accomplished by not calling `load()`, `.persist()` or `.compute()`.

#### Use the Dask dashboard

If you are using Dask distributed scheduler you can view the [dask dashboard](https://docs.dask.org/en/latest/dashboard.html?utm_source=xarray-docs) in a browser. This allows you to see memory usage, task progression and a bunch of other metrics all live.

```python
from distributed import Client

client = Client()
client
```
