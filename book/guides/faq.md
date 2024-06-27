# FAQs

(faq.cannot-log-into-hub)=

## I cannot log into the hub 😱

If you are unable to log into the hub, please check the following steps:

- \[ \] Check if you are member of the [appropriate github teams](users.categories).

If you **are not** follow these steps:

- \[ \] Did you [sign up for LEAP membership](users.membership.apply)? This will be done for you if you sign up for an event like the Momentum bootcamp!
- \[ \] Did you receive a github invite? [Here](users.membership.invite) is how to check for that.
- \[ \] Check again if they are part of the [appropriate github teams](users.categories).
- If these steps do not work, please reach out to the [](support.data_compute_team).

If you **are** member of one of the github teams, ask them to try the following steps:

- \[ \] Refresh the browser cache
- \[ \] Try a different browser
- \[ \] Restart the computer
- If these steps do not work, please reach out to the [](support.data_compute_team).

(faq.usr_dir_usage_warning)=

## I received a warning about space on my User Directory

If you get a Hub Usage Alert email, this means you are violating the User Directory storage limit (to learn why this limit exists, see read about [User Directories](hub.guide.data.user_dir)). Users who persistently violate hub usage policies may temporarily get reduced cloud access.

**Troubleshooting**

- To see which files and directories are taking up the bulk of your storage, run  `du -h --max-depth=1 ~/ | sort -h`  in Terminal. It will likely be cached files and small/medium size data files that can be removed without disrupting typical usage.
- Delete cached files, ipython checkpoints, and anything else not needed from old projects
- Move needed data files into a persistent LEAP [cloud bucket](https://leap-stc.github.io/leap-pangeo/jupyterhub.html#leap-pangeo-cloud-storage-buckets). Remember that user directories are for scripts and notebooks not datasets!

Our goal is to accomodate all community members and thus we are happy to assist users in relocating data.  If you have any concerns, please reach out to the [](support.data_compute_team). You may also consult our [Hub Data Guide](guide.hub.data) and [Data Policy](./../policies/data_policy).
