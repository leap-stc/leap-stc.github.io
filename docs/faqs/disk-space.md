# How do I manage my disk space?

- To see which files and directories are taking up the bulk of your storage, run `du -h --max-depth=1 ~/ | sort -h` in Terminal. It will likely reveal cached files and small/medium size data files that can be removed without disrupting typical usage.
- Delete cached files, ipython checkpoints, and any other unwanted files.
- If you still require more storage, it is likely that you are storing downloaded data in your user directory. We recommend storing data in a LEAP [cloud bucket](https://leap-stc.github.io/leap-pangeo/jupyterhub.html#leap-pangeo-cloud-storage-buckets) or data catalog. For more information, please consult our [](guide.data) and [](explanation.data-policy).

Our goal is to accomodate all community members and thus we are happy to assist users in relocating data. If you have any concerns, please reach out to the [](support.data_compute_team).
