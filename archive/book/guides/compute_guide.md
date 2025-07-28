# Compute Guide

These are a set of guides for using the JupyterHub Compute Environment effectively.

## Dask

Dask is a powerful tool to parallelize large-scale workflows that is tightly integrated with familiar tools such as NumPy, Pandas, and Xarray. Dask is included in the Hub's default software environment and is highly encouraged for compute-intensive tasks. For a primer on Dask, a good resource is the [Project Pythia Dask Cookbook](https://projectpythia.org/dask-cookbook/README.html). Even larger calcuations can be deployed on the Hub using [Dask Gateway](https://gateway.dask.org/). For a quick guide on how to start a Dask Cluster, consult this page from the Pangeo docs:

- <https://pangeo.io/cloud.html#dask>

(guides.compute.image.custom)=

## Creating Custom Docker Images

The LEAP-Pangeo Jupyterhub provides a selection of [software environments](reference.infrastructure.hub.software_env) that enable many workflows using the Pangeo Software stack, but often users will need to install custom dependencies. Installing dependencies every time you start a server is usually fine for testing, but can become cumbersome over the long run and makes reproducibility much harder.

The solution to this is to build a custom docker image, publish it on a registry, and then you will be able to simply paste a string into the `Image` selection box upon server startup.

Generally you have two options to generate images:

- Inherit from a community maintained existing image and add additional dependencies (**recommended**)
- Build an entirely new environment, for instance based on a conda environment file.

In most cases we recommend the first approach due to the lower maintenance burden (configurations from upstream can be adopted by a simple tag change). This guide will describe the particular steps needed to generate your own image based on one of the pangeo docker images. These instructions are based mostly on the 2i2c documentation of how to [Customize a community-maintained upstream image](https://docs.2i2c.org/admin/howto/environment/customize-image/index.html) with some modification specific to LEAP.

If you do want to build your own docker image for your project, take a look at [this template](https://github.com/2i2c-org/hub-user-image-template) and the instructions to learn how to use [repo2docker](https://github.com/jupyterhub/repo2docker) to set up CI workflows to automatically build docker images from your repository.

### Creating a repository

Create a repository from [2i2c-org/example-inherit-from-community-image](https://github.com/2i2c-org/example-inherit-from-community-image/tree/main) by clicking the `Use this Template` button in the upper right corner.

```{warning}
We highly encourage
- creating this repo under the `leap-stc` organization
- making the repo public
This will ensure all steps below work as easily as possible.
```

(guides.compute.custom_image.container_registry)=

### Set up the Container registry

We recommend hosting the image under the [`leap-stc` quay.io account](https://quay.io/user/leap-stc/). This requires actions from a member of the [](support.data_compute_team). Please open an issue titled `Set up quay.io` in your newly created repo, and add a message like this:

```
@leap-stc/data-and-compute 
can you please:
- share the quay.io secrets with this repo 
- set up a matching repo under the `leap-stc`quay.io organization
- give the robot account access to that new repo
```

```{note}
If you decide to host the image under your own account, you should follow the instructions in the [2i2c docs](https://docs.2i2c.org/admin/howto/environment/customize-image/index.html#set-up-the-github-repository-and-connect-it-to-quay-io). 
```

### Edit repository files

Now we edit the files in the repository. Chose your preferred way (e.g. in the browser, or by cloning the repositoriy locally or to the hub), and make the following changes:

- In `Dockerfile` edit the text after `"FROM"` with the image tag of the upstream image you want to build opon. The tag usually is composed as

```
<registry>/<username>/<repo_name>:<hash>
```

To inheret from the latest pangeo-notebook image you should paste:

```
quay.io/pangeo/pangeo-notebook:2024.08.18
```

```{note}
At the time of writing `2024.08.18` was the latest tag, but you should always check for the latest hash [here](https://github.com/pangeo-data/pangeo-docker-images/tags) and use that one.
```

- Now we can specify the additional requirements that should be installed on top. Edit the `environment.yml` file to your liking (you can remove the example packages from the template!).

- Edit the tests to make sure that we can import the dependencies properly. Check out the examples in the folder `image-tests`, add `.ipynb` and/or `.py` files for your added dependencies. Don't forget to remove the existing examples, since they will fail if you removed the `'otter-grader'` from `environment.yaml`!

- Finally set up the Github actions so that we can build the image in the next step. You can follow the instructions from 2i2c [here](https://docs.2i2c.org/admin/howto/environment/customize-image/index.html#build-base-image) to modify the github workflows and push changes back to Github.

```{important}
If you chose to host the image via LEAP (as recommended in [](guides.compute.custom_image.container_registry)), make sure you replace the `IMAGE_NAME` value in `.github/workflows/build.yaml` and `.github/workflows/test.yaml` with `leap-stc/<your repository name>`.
```

### Test the image

You can test the built image either on the [hub](https://docs.2i2c.org/admin/howto/environment/customize-image/index.html#test-the-custom-image-with-a-2i2c-hub) or via \[Binder\](<username>/jupyter-scipy-xarray).

If things work as expected, you are done! If you find that you are still missing dependencies, repeat the steps above to edit the `environment.yaml`, push to Github, and test until you are happy with the environment.
