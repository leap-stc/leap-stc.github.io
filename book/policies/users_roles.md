# Users and Categories

**Version 1 - 2022-05-09**

Access to the LEAP-Pangeo Hub will be categoryed based on roles in LEAP.
Membership in a particular category will be implemented via
[GitHub Teams](https://docs.github.com/en/organizations/organizing-members-into-teams)
in the [leap-stc](https://github.com/orgs/leap-stc/teams) GitHub organization.

Here we define those roles and map them to GitHub teams.

## Code of Conduct

All users of LEAP-Pangeo must abide by the LEAP Code of Conduct:

- https://docs.google.com/document/d/1eE-aYrsf_k5Ep8GB-n8hmqM_LVW-U3uzYKHJBZMvYWU

## Community Category

Any LEAP affiliate may access the community category.

Members of the Community Category may access to `Small` and `Medium` JupyterHub servers
in the LEAP-Pangeo Hub.

Membership in the Community Category is granted by adding the user to the
[`leap-pangeo-community`](https://github.com/orgs/leap-stc/teams/leap-pangeo-community)
GitHub Team.

:::{note}
The LEAP executive committee has not yet defined a process by which affiliate
status is conferred and removed.
Once this is decided, we will provide instructions on how to add / remove affiliates.
:::

## Education Category

The education category is intended for _termed access_ to LEAP-Pangeo Hub resources associated
with educational activities. Examples of educational activities include:

- Semester-long LEAP-affiliated courses
- Short bootcamps and hackathons

Members of the Community Category may access to `Small`, `Medium`, and `Large` JupyterHub servers
in the LEAP-Pangeo Hub.

Membership in the Community Category is granted by adding the user to the
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

<div class="d-flex justify-content-center m-4">
  <a class="btn btn-outline-primary" href="https://docs.google.com/forms/d/e/1FAIpQLSfqpjgc3lJaTUmh52iO5_4siZPCz1Y6oRdxMEB50qAJrVg4Hg/viewform" role="button">Submit a Proposal</a>
</div>

Once the proposal is approved, a LEAP administrator will create a GitHub sub-team
within the [`leap-pangeo-education`](https://github.com/orgs/leap-stc/teams/leap-pangeo-education)
team for the course and add the instructor as a "Maintainer".
It is the instructor's responsibility to add the course's users to this team
and remove them when the course has been concluded.

## Research Category

The education category is intended for _long-term access_ to LEAP-Pangeo resources associated
with research activities.

There are two levels to the Research Category:
- **Entry-level**: involved in LEAP research and community, have access to computing resources and storage on LEAPangeo.
Members of this Community Tier may access  `Large` JupyterHub servers
Office Space. Admission based on paragraph sent to Office Space committee.
Membership in the entry-level research category corresponds to the
[`leap-pangeo-research-entry-level`](https://github.com/orgs/leap-stc/teams/leap-pangeo-research-entry-level)
GitHub team.
- **Advanced**: RFP or supported researcher, student, postdoc, invited to annual meeting, swipe access for LEAP.
Members of this Community Tier may access `Large` and `Huge` JupyterHub servers, plus GPU access.
Membership in the entry-level research category corresponds to the
[`leap-pangeo-research-advanced`](https://github.com/orgs/leap-stc/teams/leap-pangeo-research-entry-level)
GitHub team.

### Eligibility

Anyone participating in a LEAP-sponsored research project is eligible to participate
in the Research Category.
A LEAP administrator will create a GitHub sub-team within the
[`leap-pangeo-research`](https://github.com/orgs/leap-stc/teams/leap-pangeo-research)
team for each research project and add the project PIs as "maintainer".
It is the PIs' responsibility to add and remove members from their team.

## Administrator and Developer Category

The LEAP Director of Data and Computing may grant access to other participants for
the purposes of technical development, debugging, and evaluation of the platform.

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
