The LEAP JupyterHub environment provides multiple ways to install and manage software. Choose the method that best fits your needs.

##Quick Installs (Temporary)

When you just need software for your current notebook session, you can temporarily install packages using one of the following methods. The lifetime of the installation lasts until the kernel stops. After this, it has to be re-installed.

**1. Conda-Forge**

Open the terminal inside your JupyterLab session.

Run the following code: '''conda install <package> -c conda-forge'''

**2. Mamba**

The base image does not include Mamba. However, Mamba can be useful since it is faster than Conda and can install many dependencies at once. We can install Mamba as a drop-in replacement for Conda.

Run the following code:

'''conda install -y mamba -c conda-forge'''

'''mamba install <package-name> -c conda-forge'''

**3. Pip**

Use pip when the library isn't available or up-to-date on conda-forge or you need the latest Github/PyPI release.

<<<<<<< HEAD
Run the following code: '''pip install <package-name>'''
> *Tip:* Do all Conda/Mamba installs first, then run any pip install commands. This keeps your environment consistent. 
=======
Run the following code: 'pip install <package-name>'

> *Tip:* Do all Conda/Mamba installs first, then run any pip install commands. This keeps your environment consistent.
>>>>>>> 1ebb10c2408b1fc1d8040b54f6dc209a69b58ed3

**4. Script**

If you are running the same packages every time you start a new session, it is more efficient to wrap the commands in a script.
