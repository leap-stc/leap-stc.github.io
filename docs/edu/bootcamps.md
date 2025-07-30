# Running Bootcamps

We collect all bootcamp materials in the [LEAP-Pangeo bootcamp repository](https://github.com/leap-stc/LEAP-bootcamps). Please keep all relevant information and materials in this repository to make it easier for participants to find them.

## Preparing for the bootcamp

**Past attendees have suggested that it is very helpful to have the instructor write code from scratch.**
This ensures the following:

- The lecturer takes substantially more time to write live, ensuring that others can follow along.
- The lecturer is not a superhuman, but also makes mistakes and has to debug code. This is a very important lesson for students to learn, as it is a very common part of the coding process and often not shown in lectures.
- It also allows the lecturer to explain the code as they write it, which is a very important part of the learning process.

If you are going to use a notebook pre-populated with code, please clear all outputs prior to starting the lecture by right clicking on the notebook and selecting "Clear All Outputs".

## Materials for the bootcamp

We encourage you to reuse materials from past bootcamps and adapt them to your needs. However, in doing so, please do change past materials, but rather create new folders/entries in the following places:

- Create a new folder in the "Codes" and "Lectures" folders and add your materials there. *If you are giving the same material as a previous bootcamp, please copy them.*
  - ```{warning}
      If you are using materials from other sources, like the [An Introduction to Earth and Environmental Data Science](https://earth-env-data-science.github.io/intro.html) book. Please just follow the past events structure and link to the original materials. 
    ```
- [README.md](https://github.com/leap-stc/LEAP-bootcamps/README.md) Enter a new entry under the "Events" section. This should include an [nbgitpuller](https://nbgitpuller.readthedocs.io/en/latest/) link for each notebook you work with, so participants can easily pull the materials to their hub (there is a neat tool to [generate these links](https://nbgitpuller.readthedocs.io/en/latest/link.html)). Please also use the LEAP-Pangeo Hub badge by adding code like this:
  ```
  [![Open in LEAP-Pangeo Hub](https://custom-icon-badges.demolab.com/badge/Jupyter%20Hub-Launch%20%F0%9F%9A%80-blue?style=for-the-badge&logo=leap-globe)](<generated_link>) 
  ```

## Running the bootcamp

### Prepare ahead of the event

Bootcamp instructors should make sure that they are added to the [Bootcamp Instructors Github Team](https://github.com/orgs/leap-stc/teams/bootcamp-instructors). If not please reach out to one of the github organization admins to be added. The goal should be that whoever runs the bootcamp can perform tasks that might be needed the day of (mostly likely signing up folks who either entered wrong data or did not register ahead of time).

Before the event check the following:

- Is somebody available to add new data to the Member Data Google Sheet? LEAPs Managing Director has the ability to add data or give others edit access.
- Can the instructors manually trigger the ["Parse Member Sheet and Add New Members" action](https://github.com/leap-stc/member_management/actions/workflows/read_sheet.yaml) (by clicking on "Run Workflow>Run Workflow" in the top right).
- Double Check that all participants github names are present in the [leap-pangeo-base-access](https://github.com/orgs/leap-stc/teams/leap-pangeo-base-access) team! If folks are in this team they will have access to the hub! If they are not there they either have not accepted the invite to the team (they will be shown as pending invites; tell them to accept the invite as described [here](https://leap-stc.github.io/guides/faq.html#where-is-my-invite)) or there is an issue with their github username (check their github handles in the Member Data Google Sheet for typos and completeness). We recommend that you familiarize yourself and test these steps ahead of time.

If you are anticipating many attendees to access the hub at the same time to follow along, it is useful to consult with the **[Data & Compute Team](../../support/contact)** or 2i2c beforehand to ensure a smooth presentation.

### Troubleshooting during the event

If students have trouble signing in to the hub, please refer to the **FAQ** for troubleshooting steps.

## After the event

Please add your versions of the filled notebooks to a subfolder named `filled_notebooks` in the event folder you created above, so participants and future instructors can access them later.
