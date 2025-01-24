(reference.membership)=

# Membership

Access to LEAP-Pangeo is managed via membership tiers and associated github organization teams.

## Code of Conduct

All users of LEAP-Pangeo must abide by the [LEAP Code of Conduct](https://leap.columbia.edu/wp-content/uploads/2023/05/LEAP-Code-of-Conduct_V1_October-2022.pdf).

(reference.membership.tiers)=

## Membership Tiers

The membership tiers are listed in ascending order of access/privileges.

- **PUBLIC MEMBERSHIP(Please note: Public Membership will be live soon.)**  Receive weekly LEAP newsletter and invitations to LEAP events. Access to LEAP Pangeo that includes limited performance notebooks, read only access to all LEAP storage, and write access to our scratch bucket, where everything is deleted after 7 days.
- **EDUCATION MEMBERSHIP**  Open to participants in LEAP Education programs (e.g., Bootcamp, academic courses, and summer programs). Receive weekly LEAP newsletter and invitations to LEAP events. Access to LEAP Pangeo compute and storage for duration of education program.
- **RESEARCH MEMBERSHIP** Open to researchers who have been referred by a LEAP scientist. Receive weekly LEAP newsletter and invitations to LEAP events. Access to LEAP Pangeo compute and storage for six (6) months (renewal applications may be submitted at the conclusion of six months).
- **LEAP-FUNDED RESEARCHE MEMBERSHIP**. Open to researchers who receive LEAP funding. Receive weekly LEAP newsletter and invitations to LEAP events. Access to LEAP Pangeo compute and storage for duration of time with LEAP.

For more info see [the main LEAP website](https://leap.columbia.edu/research-home/leap-pangeo/)

### Administrator and Developer Category

The LEAP Manager of Data and Computing may grant access to other participants for
the purposes of technical development, debugging, and evaluation of the platform. These members will be added to the [`leap-pangeo-full-access`](https://github.com/orgs/leap-stc/teams/leap-pangeo-full-access) team to have full access to all resources.

(reference.membership.team-resources)=

## Github Teams and Resources

The access to the JupyterHub is implemented via [Github Teams](https://docs.github.com/en/organizations/organizing-members-into-teams/about-teams) in the [leap-stc](https://github.com/orgs/leap-stc/teams) GitHub organization. Each Membership tier maps to a single Github Team which determines the resources available to the user.

| Tier                     | Github Team                                                                                   | Resources Available                                                                                                                            | Membership Valid for |
| ------------------------ | --------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- | -------------------- |
| **PUBLIC**               | [leap-pangeo-public-access](https://github.com/orgs/leap-stc/teams/leap-pangeo-public-access) | 🚧                                                                                                                                             | 🚧                   |
| **EDUCATION**            | [leap-pangeo-base-access](https://github.com/orgs/leap-stc/teams/leap-pangeo-base-access)     | Access to [storage](reference.infrastructure.buckets) and computing resource up to 4 cores and 32GB RAM on JupyterHub servers.                 | 🚧                   |
| **RESEARCH**             | [leap-pangeo-full-access](https://github.com/orgs/leap-stc/teams/leap-pangeo-full-access)     | Access to [storage](reference.infrastructure.buckets) and computing resource up to 16 cores and 128GB RAM on JupyterHub servers + GPU options. | 🚧                   |
| **LEAP-FUNDED RESEARCH** | [leap-pangeo-full-access](https://github.com/orgs/leap-stc/teams/leap-pangeo-full-access)     | Access to [storage](reference.infrastructure.buckets) and computing resource up to 16 cores and 128GB RAM on JupyterHub servers + GPU options. | 🚧                   |

(users.membership.apply)=

## Applying for membership

To become a LEAP member please use the [Application Form](https://forms.gle/RpeaMZh5btTdZtzu8).

Once you have applied for membership, it will take a few days until you can be approved. Please watch out for email from LEAP that contains instructions on how to proceed.

## Termination of Access

Users who violate usage policies will have their access suspended pending investigation.
The LEAP Director of Data and Computing decides if a policy has been violated and
may suspend or terminate access to LEAP-Pangeo at any time.

## Offboarding Process

Users may also be transferred from e.g. the Education Category to the Community Category
when their termed access ends.
Removing a user from the corresponding team is sufficient to disable their access
to those resources.
Removing a user from the `leap-pangeo-users` group entirely will disable their access
completely.
An automated process will delete user data from the hub one month after a user
is removed from the `leap-pangeo-users` group.

(reference.member_sign_up)=

## Member Sign Up Procedure

![](../images/member_sign_up_schematic.png)

All data relevant to LEAP membership is centrally processed in a spreadsheet with access limited to LEAP staff. To allow LEAP members access to the JupyterHub (and the associated storage and compute resources; all managed by 2i2c) the [](support.data_compute_team) maintains a set of [Github Actions](https://github.com/leap-stc/member_management/actions/workflows/read_sheet.yaml)(the member management repository is private) that parse the relevant data like github usernames and membership expiration, and add users to the appropriate [teams](reference.membership.team-resources).
For normal operations users should [apply](users.membership.apply) and wait a few days until they are approved, entered, and signed up for access.
For urgent situations (like events, classes) that require expedited sign up please refer to [](guide.team.admin.member_signup_troubleshooting).
