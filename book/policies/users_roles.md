# Users and Tiers

Access to the LEAP-Pangeo JupyterHub will be tiered based on roles in LEAP.
Membership in a particular tier will be implemented via
[GitHub Teams](https://docs.github.com/en/organizations/organizing-members-into-teams)
in the [leap-stc](https://github.com/orgs/leap-stc/teams) GitHub organization.

Here we define those roles and map them to GitHub teams.

## Community Tier

Any LEAP affiliate may access the community tier.

Members of the Community Tier may access to `Small` and `Medium` JupyterHub servers
in the LEAP-Pangeo JupyterHub.

Membership in the Community Tier is granted by adding the user to the
[`leap-pangeo-users`](https://github.com/orgs/leap-stc/teams/leap-pangeo-users)
GitHub Team.
(All subsequent teams are defined sub-teams of this team.)

:::{note}
The LEAP executive committee has not yet defined a process by which affiliate
status is conferred and removed.
This process should include adding / removing them from this GitHub Team.
:::

## Education Tier

The education tier is intended for _termed access_ to LEAP-Pangeo resources associated
with educational activities. Examples of educational activities include:

- Semester-long LEAP-affiliated courses
- Short bootcamps and hackathons

Members of the Community Tier may access to `Small`, `Medium`, and `Large` JupyterHub servers
in the LEAP-Pangeo JupyterHub.

Membership in the Community Tier is granted by adding the user to the
[`leap-pangeo-education`](https://github.com/orgs/leap-stc/teams/leap-pangeo-education)
GitHub Team.
Additional sub-teams may be created within this team to organize students into
specific courses, bootcamps, etc.
It is the instructor's responsibility to add / remove the user from this team.

## Research Tier

he education tier is intended for _long-term access_ to LEAP-Pangeo resources associated
with research activities.
Members of the Community Tier may access to `Small`, `Medium`, `Large`, and `Huge`
JupyterHub servers. They also have the option to attach GPUs to their server.

Membership in the Community Tier is granted by adding the user to the
[`leap-pangeo-research`](https://github.com/orgs/leap-stc/teams/leap-pangeo-research)
GitHub Team.
Additional sub-teams may be created within this team to organize scientists into
specific research projects.
It it the PI's responsibility to add / remove users from this team.

## Offboarding Process

Users may also be transferred from e.g. the Education Tier to the Community Tier
when their termed access ends.
Removing a user from the corresponding team is sufficient to disable their access
to those resources.
Removing a user from the `leap-pangeo-users` group entirely will disable their access
completely.
An automated process will delete user data from the hub one month after a user
is removed from the `leap-pangeo-users` group.
