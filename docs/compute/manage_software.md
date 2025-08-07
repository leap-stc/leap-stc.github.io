# Managing Software

The LEAP JupyterHub environment provides multiple ways to install and manage software. Choose the method that best fits your needs.

## Quick Installs (Temporary)

When you just need software for your current notebook session, you can temporarily install packages using one of the following methods. The lifetime of the installation lasts until the kernel stops. After this, it has to be re-installed.

!!! note

    Each shell command listed can also be run inside an active Jupyter Notebook by prefixing them with an exclamation mark (i.e. running `!pip install numpy` in a code cell).

**1. Conda (or Mamba)**

Open the terminal inside your JupyterLab session.

Run the following code: `conda install <package>`

!!! tip

`mamba` is available in the base image and is a faster alternative to `conda`. Use the command: `mamba install <package-name>`

**2. Pip**

Use pip if the package isn't available or up to date on conda-forge, or if you are installing directly from Github or PyPI.
Run the following code: `pip install <package-name>`

!!! tip

Do all Conda/Mamba installs first, then run any pip install commands. This keeps your environment consistent.

**3. Script**

If you are installing the same packages every time you start a new session, it is more efficient to wrap the commands in a script. This is especially useful for workshops or group work.

Create a .sh file with the following skeletal code:

```bash
#!/bin/bash
conda install -y <package> <package> 
pip install <package>
echo "Environment ready!"
```

Then in the terminal run the following code to make the script executable: `chmod +x setup-env.sh`

You can now execute the script in your terminal by the following: `./<filename>.sh`

## Persistant Installs (Docker Image)

In some cases, a custom Docker image is required to ensure that software packages are installed persistently across kernel restarts. Unlike runtime installations, which are temporary and must be re-run each session, a custom image guarantees that packages are available every time a user starts a new server.

Use a custom image for the following scenarios:

- When multiple users need the same libraries
- When setting up workshops, bootcamps, or other workflows
- When packages take a long time to install or have complex dependencies
- When JupyterLab extensions must be pre-built to function properly (ex: AmazonQ, classic labextensions, etc)

If you find yourself installing the same packages repeatedly, or if runtime installs are unreliable or time-consuming, consider moving to a custom image for a more stable and efficient experience.

For instructions on creating and managing a custom image, see the [2i2c hub-user-image guide](https://docs.2i2c.org/admin/howto/environment/)

### How to use a custom image in the LEAP Hub:

1. Go to the **Start Server** page in JupyterHub.
    (include link to screenshot here)

1. Select **Other...** from the **Image options** dropdown. Then enter the link to the custom image in the form of `<registry>/<username>/<repo_name>:<git-commit-hash>`
    (include link to screenshot here)

1. Launch the server - your packages will be pre-installed.
