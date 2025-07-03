(guide.vscode)=

# Using VSCode with the Hub

## Pre-requisites

1. LEAP Pangeo Access

You must have the ability to start a server on LEAP Pangeo. More information about membership can be found in the [Membership](https://leap-stc.github.io/reference/membership.html#users-membership-apply) page.

1. Websocat

[websocat](https://github.com/vi/websocat) must be installed on the client machine.
`pip install websocat` works on Mac OS, and pre-built binaries [are available](https://github.com/vi/websocat/releases)
for all other operating systems.

1. VSCode

[VSCode](https://code.visualstudio.com/download) must be installed on the client machine.

1. VSCode Extensions

Find and install the Remote-SSH and Jupyter extensions on the VSCode Extensions Marketplace.

<img width="268" alt="Image" src="https://github.com/user-attachments/assets/6dc5d51c-3389-4e2d-ab92-4ee75c981aa5" />
<img width="272" alt="Image" src="https://github.com/user-attachments/assets/67f748fc-7ae0-4cc0-a1b5-a6a4ce8b67e1" />

Remote-SSH will be used to connect to your LEAP Pangeo server, while the Jupyter extension will be used to open .ipynb notebooks and manage the kernel that runs your code on the remote server.

## [Web Browser] Start your server

This setup only works after you start your JupyterHub server. So, start your server!
Please keep in mind that even after setting up Remote-SSH on your VSCode, you would have to start your server on [LEAP Pangeo](https://leap.2i2c.cloud/hub/home) before attempting to connect to the server.

## [Web Browser] Create a JupyterHub Token

We will need to create a JupyterHub token for authentication.

1. Go to the JupyterHub control panel. You can access it via `File -> Hub control panel` in
   JupyterLab, or directly going to [https://leap.2i2c.cloud/hub/token](https://leap.2i2c.cloud/hub/token).

1. In the top bar, select **Token**.

1. Create a new Token, and keep it safe. **Treat this like you would treat a password to your
   JupyterHub instance**! It is recommended you set an expiry date for this.

## [Local Terminal] Setup your local `~/.ssh/config`

We will set up our ssh config file to tell `ssh` how to connect to our JupyterHub. Add
an entry that looks like this to the end of your `~/.ssh/config` file (create it if it does not exist).

```
Host leap.2i2c.cloud
    User jovyan
    ProxyCommand websocat --binary -H='Authorization: token <YOUR-JUPYTERHUB-TOKEN>' asyncstdio: wss://%h/user/<YOUR-JUPYTERHUB-USERNAME>/sshd/
```

replace:

- `<YOUR-JUPYTERHUB-TOKEN>` with the token you generated earlier
- `<YOUR-JUPYTERHUB-USERNAME>` with your jupyterhub username

## [Local Terminal + Github] Setup your ssh key and save the public key on Github

1. Generate key on local terminal:
   `ssh-keygen -t ed25519 -C "your_email@example.com"`
1. Add key to ssh-agent:
   `eval "$(ssh-agent -s)"`
   `ssh-add  #~/.ssh/id_ed25519`
1. Copy and paste the key to your Github account. Use following command to show the key on your local machine:
   `cat ~/.ssh/id_ed25519.pub` - This is the public key, do not expose your private key `id_ed25519` - the private key should not be shared.
   In [Github key settings](https://github.com/settings/keys), paste the entire output from previous command (including email address) into the “Key” box, and save.

## [Web Browser] Setup ssh keys on your JupyterHub server

There are two levels of authentication - your JupyterHub token, as well as your ssh keys (public / private).

1. After you start your JupyterHub server, open a terminal in JupyterLab

1. Run the following commands:

   ```bash
   mkdir -p ~/.ssh
   wget https://github.com/<YOUR-GITHUB-USERNAME>.keys -O ~/.ssh/authorized_keys
   chmod 0600 ~/.ssh/authorized_keys
   ```

   replacing `<YOUR-GITHUB-USERNAME>` with your github username.

1. Verify that authorized_keys contains your key by running `cat ~/.ssh/authorized_keys` on the JupyterLab Terminal

1. If it is empty or nonexistent, then create the file authorized_keys and paste your ssh public key (`cat ~/.ssh/id_ed25519.pub`, from local terminal) and save file.

With that, we are ready to go!

## [Local Terminal] Try `ssh`-ing into your JupyterHub!

After all this is setup, you're now able to ssh in! On your local terminal, try:

```
ssh leap.2i2c.cloud
```

and it should just work! If configured correctly, `leap.2i2c.cloud` should not ask you for a password.

```{admonition} Debugging Help
---
class: important
---
If you get the error `ssh: connect to host leap.2i2c.cloud port 22: Operation timed out`, then check you have installed websocat by running  `run pip install websocat` and confirm that ther config file in the correct directory, `~/.ssh/config`
```

If the CLI asks for a password, please verify that your access token and public keys are valid and consistent across platforms and try the previous steps again. Keep in mind this test has to only work once, and it is not necessary to ssh into JupyterHub via CLI once you confirm this works once.

## [VSCode] Connect to LEAP Pangeo on VSCode

Launch VSCode, click on the top search bar to open the Command Palette (cmd + shift + P on Mac)

1. Enter `>Remote-SSH: Add New SSh Host`

<img width="636" alt="Image" src="https://github.com/user-attachments/assets/5aaaf786-5d27-4cc3-a290-889a06e3ab4c" />

1. Enter `ssh leap.2i2c.cloud`

<img width="656" alt="Image" src="https://github.com/user-attachments/assets/ef8a0205-ff73-42b3-b60f-497152b41ac7" />

1. Click on the first option, /Users/\<your_local_username>/.ssh/config

<img width="644" alt="Image" src="https://github.com/user-attachments/assets/e3e6db0d-9579-41d3-9678-b6f71c0875db" />

1. Follow further prompts until you are connected. You should see this on your top bar once you are connected.

<img width="620" alt="Image" src="https://github.com/user-attachments/assets/0a8a6698-1c69-41cf-9b3b-b13ac1450409" />

1. Click on "Open Folder" and select the home directory. You should be able to access your LEAP-Pangeo Notebooks remotely now.

<img width="329" alt="Image" src="https://github.com/user-attachments/assets/15620385-784a-427a-8d6c-cd099e1b3017" />

## [VSCode] Write / execute your code via a remote kernel

1. Once you open a Notebook, you will find a selection menu for a kernel on the top-right.

<img width="1471" alt="Image" src="https://github.com/user-attachments/assets/f5a18a77-fd3d-4104-8f84-3ca2d5b9374a" />

1. Select the option "Python Environments"

<img width="610" alt="Image" src="https://github.com/user-attachments/assets/867fa947-811d-4a52-b147-61a1a2b88bc6" />

1. Selected the recommended `conda (Python 3.x.x)` kernel

<img width="610" alt="Image" src="https://github.com/user-attachments/assets/772746e7-4eba-49ba-9a89-465b1b19a140" />

You should now be good to execute code via remote kernel on the LEAP server from local VSCode!

## [VSCode] Re-connecting to Remote SSH After Setup

1. Type `>Remote-SSH: Connect to Host` into the top search bar.

<img width="755" alt="Image" src="https://github.com/user-attachments/assets/5a015483-05bd-4a43-b449-eab593cb0687" />

1. Select `leap.2i2c.cloud`. Keep in mind this action will only work if the [2i2c Server](https://leap.2i2c.cloud/hub/home) is already active.

<img width="764" alt="Image" src="https://github.com/user-attachments/assets/a8d95afd-e0ed-41fe-8020-c840516e4dc1" />

1. Repeat the steps from _"Write / execute your code via a remote kernel"_

Contributors to this documentation:

1. Yuvi [https://github.com/yuvipanda], [jupyter-sshd-proxy](https://github.com/yuvipanda/jupyter-sshd-proxy/blob/main/README.md):
1. Joe Ko [jk4730@columbia.edu], clarifications on how to setup the local public key
1. Sungjoon Park [sp4050@columbia.edu], whom you can contact for questions regarding this workflow 