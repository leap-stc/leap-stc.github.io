# Hub Usage Alert

## I received a warning about space on my User Directory

If you get a Hub Usage Alert email, this means you are violating the User Directory storage limit (to learn why this limit exists, see read about our [Data Policy](../../data/data_policy). Remember that user directories are for scripts and notebooks not datasets! Users who persistently violate hub usage policies may get their cloud access temporarily restricted.

**Troubleshooting**

- To see which files and directories are taking up the bulk of your storage, run `du -h --max-depth=1 ~/ | sort -h` in Terminal. It will likely reveal cached files and small/medium size data files that can be removed without disrupting typical usage.
- Delete cached files, ipython checkpoints, and any other unwanted files.
- If you still require more storage, it is likely that you are storing downloaded data in your user directory. We recommend storing data in a LEAP cloud bucket or data catalog. For more information, please consult our [data locations](../data/data_locations.md).

Our goal is to accomodate all community members, so we are happy to assist users in relocating data. If you have any concerns, please reach out to the [Data and Compute Team](../support/contact.md).
