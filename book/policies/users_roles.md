# Users and Tiers

Access to the LEAP-Pangeo Hub will be tiered based on roles in LEAP.
Membership in a particular tier will be implemented via
[GitHub Teams](https://docs.github.com/en/organizations/organizing-members-into-teams)
in the [leap-stc](https://github.com/orgs/leap-stc/teams) GitHub organization.

Here we define those roles and map them to GitHub teams.

## Community Tier

Any LEAP affiliate may access the community tier.

Members of the Community Tier may access to `Small` and `Medium` JupyterHub servers
in the LEAP-Pangeo Hub.

Membership in the Community Tier is granted by adding the user to the
[`leap-pangeo-community`](https://github.com/orgs/leap-stc/teams/leap-pangeo-community)
GitHub Team.

:::{note}
The LEAP executive committee has not yet defined a process by which affiliate
status is conferred and removed.
Once this is decided, we will provide instructions on how to add / remove affiliates.
:::

## Education Tier

The education tier is intended for _termed access_ to LEAP-Pangeo Hub resources associated
with educational activities. Examples of educational activities include:

- Semester-long LEAP-affiliated courses
- Short bootcamps and hackathons

Members of the Community Tier may access to `Small`, `Medium`, and `Large` JupyterHub servers
in the LEAP-Pangeo Hub.

Membership in the Community Tier is granted by adding the user to the
[`leap-pangeo-education`](https://github.com/orgs/leap-stc/teams/leap-pangeo-education)
GitHub Team.
Additional sub-teams will be created within this team to organize students into
specific courses, bootcamps, etc.

### Eligibility

Course instructors who meet one of the following criteria are eligible to request
access for their class (including self, co-instructor(s), TAs, students, evaluators, etc):
- LEAP senior personnel or advanced research member
- Full-time faculty at LEAP Institution (Columbia, NYU, UMN, UCI)
- Full-time faculty who participated in a LEAP’s "train the trainer" workshop

### Proposal Process

Course instructors may propose to the LEAP-Pangeo Hub for their upcoming course by submitting
a short proposal to the LEAP’s Convergence Subcommittee.
Instructors are required to submit their proposal at least 30 days before the participants will require.
The proposal should provide the following information.
- Instructor's GitHub username
- Basic information about the course (institution, department/program, student population, class size, course dates)
- Basic information about the Instructor (name, affiliation, research interests)
- Confirmation of available administrative support for using the LEAP-Pangeo Hub for the class (setting up user account, monitoring use, etc)
- Anticipated usage level of LEAP-Pangeo Hub, including
  - Number of user hours / week
  - Types of virtual machines to be used (`Small`, `Medium`, or `Large`)

<a href="link to form"><button type="button">Submit a Proposal</button></a>

Once the proposal is approved, a LEAP administrator will create a GitHub sub-team
within the [`leap-pangeo-education`](https://github.com/orgs/leap-stc/teams/leap-pangeo-education)
team for the course and add the instructor as a "Maintainer".
It is the instructor's responsibility to add the course's users to this team
and remove them when the course has been concluded.

## Research Tier

The education tier is intended for _long-term access_ to LEAP-Pangeo resources associated
with research activities.
Members of the Community Tier may access to `Small`, `Medium`, `Large`, and `Huge`
JupyterHub servers. They also have the option to attach GPUs to their server.
Membership in the Research tier corresponds to the
[`leap-pangeo-research`](https://github.com/orgs/leap-stc/teams/leap-pangeo-research)
GitHub team.

### Eligibility

Anyone participating in a LEAP-sponsored research project is eligible to participate
in the Research Tier.
A LEAP administrator will create a GitHub sub-team within the
[`leap-pangeo-research`](https://github.com/orgs/leap-stc/teams/leap-pangeo-research)
team for each research project and add the project PIs as "maintainer".
It is the PIs' responsibility to add and remove members from their team.

## Offboarding Process

Users may also be transferred from e.g. the Education Tier to the Community Tier
when their termed access ends.
Removing a user from the corresponding team is sufficient to disable their access
to those resources.
Removing a user from the `leap-pangeo-users` group entirely will disable their access
completely.
An automated process will delete user data from the hub one month after a user
is removed from the `leap-pangeo-users` group.
