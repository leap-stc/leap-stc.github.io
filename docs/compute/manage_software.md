The LEAP JupyterHub environment provides multiple ways to install and manage software. Choose the method that best fits your needs.

## Quick Installs (Temporary)

When you just need software for your current notebook session, you can temporarily install packages using one of the following methods. Each shell command listed can also be run inside an active Jupyter Notebook if prefixed with an exclamation mark (i.e. running `!pip install numpy` in a code cell). The lifetime of the installation lasts until the kernel stops. After this, it has to be re-installed.

**1. Conda-Forge**

Open the terminal inside your JupyterLab session.

Run the following code: `conda install <package> -c conda-forge`

**2. Mamba**

The base image does not include Mamba. However, Mamba can be useful since it is faster than Conda and can install many dependencies at once. We can install Mamba as a drop-in replacement for Conda.

Run the following code:

`conda install -y mamba -c conda-forge`

`mamba install <package-name> -c conda-forge`

**3. Pip**

Use pip when the library isn't available or up-to-date on conda-forge or you need the latest Github/PyPI release.
Run the following code: `pip install <package-name>`

> *Tip:* Do all Conda/Mamba installs first, then run any pip install commands. This keeps your environment consistent.

**4. Script**

If you are running the same packages every time you start a new session, it is more efficient to wrap the commands in a script. This is useful for workshops or group work.

Create a .sh file with the following skeletal code:

\\n#!/bin/bash
conda install -y <package> <package> -c conda-forge
pip install <package>
echo "Environment ready!"\\n

Then in the terminal run the following code to make the script executable: `chmod +x setup-env.sh`

You can now execute the script in your terminal by the following: `./<filename>.sh`

## Persistant Installs (Docker Image)

In some cases, a custom Docker image is required to ensure that software packages are installed persistently across kernel restarts. Unlike runtime installations, which are temporary and must be re-run each session, a custom image guarantees that packages are available every time a user starts a new server.

Use a custom image when:

- The same libraries are needed in every session across users
- Workshops, bootcamps, or other workflows
- Packages require long install times or complex system dependencies
- JupyterLab extensions must be pre-built to function properly (ex: AmazonQ, classic labextensions, etc)

If you find yourself installing the same packages repeatedly, or if runtime installs are unreliable or time-consuming, consider moving to a custom image for a more stable and efficient experience.

For instructions on creating and managing a custom image, see the [2i2c hub-user-image guide](https://docs.2i2c.org/admin/howto/environment/)
