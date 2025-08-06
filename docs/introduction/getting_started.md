# Getting Started With JupyterHub

## Gaining Access

To gain access to the Hub please apply via our [Application Form](https://forms.gle/RpeaMZh5btTdZtzu8). Access to the JupyterHub is implemented via [Github Teams](https://docs.github.com/en/organizations/organizing-members-into-teams/about-teams) in the [leap-stc](https://github.com/orgs/leap-stc/teams) GitHub organization. Each membership tier is associated with a Github Team and determines the resources available to the user:

| Tier                      | Github Team                                                                                   | Resources Available                                                                                                                            | Intended for                                                                                     |
| ------------------------- | --------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| **PUBLIC** *Not live yet* | [leap-pangeo-public-access](https://github.com/orgs/leap-stc/teams/leap-pangeo-public-access) | Read access to LEAP storage, write access to scratch bucket, and access to LEAP Pangeo that includes limited performance notebooks.            | Anyone!                                                                                          |
| **EDUCATION**             | [leap-pangeo-base-access](https://github.com/orgs/leap-stc/teams/leap-pangeo-base-access)     | Access to [storage](reference.infrastructure.buckets) and computing resource up to 4 cores and 32GB RAM on JupyterHub servers.                 | Participants in LEAP Education programs (e.g., Bootcamp, academic courses, and summer programs). |
| **RESEARCH**              | [leap-pangeo-full-access](https://github.com/orgs/leap-stc/teams/leap-pangeo-full-access)     | Access to [storage](reference.infrastructure.buckets) and computing resource up to 16 cores and 128GB RAM on JupyterHub servers + GPU options. | Researchers who have been referred by a LEAP scientist. Six month renewal application required.  |
| **LEAP-FUNDED RESEARCH**  | [leap-pangeo-full-access](https://github.com/orgs/leap-stc/teams/leap-pangeo-full-access)     | Access to [storage](reference.infrastructure.buckets) and computing resource up to 16 cores and 128GB RAM on JupyterHub servers + GPU options. | Researchers receiving LEAP funding.                                                              |

Once you have applied for membership, it will take a few days until you can be approved. Please watch out for email from LEAP that contains instructions on how to proceed.
!!! note
It is very common for users to not realize that they have already received an invitation to join the `leap-stc` Github Organization! If you are unsure whether your membership has gone through, please consult our [FAQs](../technical-reference/faqs.md)

## Logging In

1. 👀 Navigate to <https://leap.2i2c.cloud/> and click the big orange button that says "Log in to continue"
1. 🔐 You will be prompted to authorize a GitHub application. Say "yes" to everything.
   Note you must belong to the appropriate GitHub team in order to access the hub.
   See [](reference.membership.team-resources) for more information.
1. 📠 You will redirect to a screen with server configuration options. Several drop down menus enable users to choose their environment image (default is "Base Pangeo Notebook") and compute resources (CPU, GPU if enabled).
   ![Server Options](../assets/hub_menu.png)
1. 🕥 After clicking "Start", wait for your server to start up. It can take up to few minutes.
   !!! note
   Depending on your [membership](reference.membership.tiers) you might see different options (e.g. GPU might be hidden).

You have to make 3 choices here:

- The machine type (Choose between "CPU only" or "GPU" if available)
  **⚠️The GPU images should be used only when needed to accelerate model training.**
- The software environment ("Image"), each of which is based upon [docker images](https://www.digitalocean.com/community/tutorials/the-docker-ecosystem-an-introduction-to-common-components) which you can run on other machines (like your laptop or an HPC cluster) for better reproducibility. If unfamiliar, you can think of docker images as more powerful conda environments bundling everything you need to do your science!
- The node share. These are shared resources, and you should try to use the smallest image you need. You can easily start up a new server with a larger share if you find your work to be limited by CPU/RAM.

If unsure which options to choose, check out our guide on [scaling compute](../compute/compute_scaling.md).

## JupyterLab

After your server fires up, you will be dropped into a JupyterLab environment. From here you can open your own shell terminals, write scripts, and create iPython Notebooks! You also have seamless access to multiple [cloud storage options](../data/data_locations.md) adjacent to your compute resources.

If you are new to JupyterLab, you might want to peruse the [user guide](https://jupyterlab.readthedocs.io/en/stable/user/interface.html).
