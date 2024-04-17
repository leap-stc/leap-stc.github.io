# Running LEAP bootcamps

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
:class: important
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

If students have trouble signin into the hub, please refer to the [FAQ](faq.cannot-log-into-hub) for troubleshooting steps.

## Debrief

Please add your versions of the filled notebooks to a subfolder named `filled_notebooks` in the eventfolder you created above, so participants can access them later.
