---
title: Hosting a Bootcamp
---

# Running Bootcamps

We collect all bootcamp materials in the [LEAP-Pangeo bootcamp repository](https://github.com/leap-stc/LEAP-bootcamps). Please keep all relevant information and materials in this repository to make it easier for participants to find them.

## Preparing the bootcamp

We encourage you to reuse materials from past bootcamps and adapt them to your needs. **Past event have suggested that it is extremely helpful to have the instructor write down code live, so participants can follow along.**
This ensures the following:

- The lecturer substantially more time to write live, ensuring that others can follow along
- It also demonstrates that the lecturer is not a superhuman, but also makes mistakes and has to debug code. This is a very important lesson for students to learn, as it is a very common part of the coding process and often not shown in lectures.
- It also allows the lecturer to explain the code as they write it, which is a very important part of the learning process.

Ideally you will use the same materials as the participants (see below for how to organize them), and clear all outputs prior to starting the lecture by right clicking on the notebook and selecting "Clear All Outputs".

> **Of course you are not expected to remember all the code. Ideally you have a second laptop/tablet to use as a cheat sheet during the presentation.**

```{admonition} Pretty please
---
class: important
---
It would be great if you can commit your filled notebooks after the lecture, so participants can access them later. See below for where to commit them.
```

## Materials for the bootcamp

You can reuse the materials for your own bootcamps or modify them to your needs. Please do not change the past materials, and instead create new folders/entries in the following places:

- Create a new folder in the "Codes" and "Lectures" folders and add your materials there. *If you are giving the same material as a previous bootcamp, please copy them.*
  - ```{warning}
      If you are using materials from other sources, like the [An Introduction to Earth and Environmental Data Science](https://earth-env-data-science.github.io/intro.html) book. Please just follow the past events structure and link to the original materials. 
    ```
- [README.md](https://github.com/leap-stc/LEAP-bootcamps/README.md) Enter a new entry under the "Events" section. This should include an [nbgitpuller](https://nbgitpuller.readthedocs.io/en/latest/) link for each notebook you work with, so participants can easily pull the materials to their hub (there is a neat tool to [generate these links](https://nbgitpuller.readthedocs.io/en/latest/link.html)). Please also use the LEAP-Pangeo Hub badge by adding code like this:
  ```
  [![Open in LEAP-Pangeo Hub](https://custom-icon-badges.demolab.com/badge/Jupyter%20Hub-Launch%20%F0%9F%9A%80-blue?style=for-the-badge&logo=leap-globe)](<generated link>) 
  ```
  > Add the generated link to the `<>` brackets.

## Running the bootcamp

### Prepare ahead of the event

Bootcamp instructors should make sure that they are added to the [Bootcamp Instructors Github Team](https://github.com/orgs/leap-stc/teams/bootcamp-instructors). If not please reach out to one of the github organization admins to be added. The goal should be that whoever runs the bootcamp can perform tasks that might be needed the day of (mostly likely signing up folks who either entered wrong data or did not register ahead of time).

Before the event check the following:

- Is somebody available to add new data to the Member Data Google Sheet? LEAPs Managing Director has the ability to add data or give others edit access.
- Can the instructors manually trigger the ["Parse Member Sheet and Add New Members" action](https://github.com/leap-stc/member_management/actions/workflows/read_sheet.yaml) (by clicking on "Run Workflow>Run Workflow" in the top right).
- Double Check that all participants github names are present in the [leap-pangeo-base-access](https://github.com/orgs/leap-stc/teams/leap-pangeo-base-access) team! If folks are in this team they will have access to the hub! If they are not there they either have not accepted the invite to the team (they will be shown as pending invites; tell them to accept the invite as described [here](https://leap-stc.github.io/guides/faq.html#where-is-my-invite)) or there is an issue with their github username (check their github handles in the Member Data Google Sheet for typos and completeness). Find a more detailed guide for troubleshooting [here](guide.team.admin.member_signup_troubleshooting). We recommend that you familiarize yourself and test these steps ahead of time.

If you are anticipating a large demand for an event it might be useful to consult with the [](support.data_compute_team) or 2i2c beforehand.

### Troubleshooting during the event

If students have trouble signin into the hub, please refer to the [FAQ](faq.cannot-log-into-hub) and [](guide.team.admin.member_signup_troubleshooting) for troubleshooting steps.

## Debrief

Please add your versions of the filled notebooks to a subfolder named `filled_notebooks` in the eventfolder you created above, so participants can access them later.
