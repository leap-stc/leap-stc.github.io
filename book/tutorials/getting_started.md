(tutorial.getting_started)=

# Getting Started

To get started using the hub, check out this video by [James Munroe](https://github.com/jmunroe) from [2i2c](https://2i2c.org) explaining the architecture.

<iframe width="560" height="315" src="https://www.youtube.com/embed/RKXWxtNqWKw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

(hub.access)=

## How can I get access to the Hub

Only LEAP members will be able to access the hub. The tier of access is directly determined by membership in the corresponding [Github Team](reference.membership.team-resources). Please [become a member](users.membership.apply) and make sure you accept the [invitation on github](faq.where_is_my_invite) before proceeding.

## Hub Usage

This is a rough and ready guide to using the Hub.
This documentation will be expanded as we learn and evolve.
Feel free to [edit it yourself](https://github.com/leap-stc/leap-stc.github.io/blob/main/book/leap-pangeo/jupyterhub.md) if you have suggetions for improvement!

(hub:server:login)=

### Logging In

1. 👀 Navigate to <https://leap.2i2c.cloud/> and click the big orange button that says "Log in to continue"
1. 🔐 You will be prompted to authorize a GitHub application. Say "yes" to everything.
   Note you must belong to the appropriate GitHub team in order to access the hub.
   See [](reference.membership.team-resources) for more information.
1. 📠 You will redirect to a screen with the following options.

<img width="410" alt="image" src="https://github.com/leap-stc/leap-stc.github.io/assets/14314623/088946a1-896f-4ff8-af91-8107c9f14cfd">

> Note: Depending on your [membership](reference.membership.tiers) you might see additional options (e.g. for GPU machines)

You have to make 3 choices here:

- The machine type (Choose between "CPU only" or "GPU" if available)
  **⚠️The GPU images should be used only when needed to accelerate model training.**
- The software environment ("Image"). Find more info in the [Software Environment Section](reference.infrastructure.hub.software_env).
- The node share. These are shared resources, and you should try to use the smallest image you need. You can easily start up a new server with a larger share if you find your work to be limited by CPU/RAM

4. 🕥 Wait for your server to start up. It can take up to few minutes.

#### GPUs

Certain members (see [](reference.membership.team-resources)) have access to server instance with GPU. Currently the GPUs are [Nvidia T4](https://www.nvidia.com/en-us/data-center/tesla-t4/) models. To check what GPU is available on your server you can use [`nvidia-smi`](https://developer.nvidia.com/nvidia-system-management-interface) in the terminal window. You should get output similar to this:

```shell

   nvidia-smi
   
   +-----------------------------------------------------------------------------+
   | NVIDIA-SMI 510.47.03    Driver Version: 510.47.03    CUDA Version: 11.6     |
   |-------------------------------+----------------------+----------------------+
   | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
   | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
   |                               |                      |               MIG M. |
   |===============================+======================+======================|
   |   0  Tesla T4            Off  | 00000000:00:04.0 Off |                    0 |
   | N/A   41C    P8    11W /  70W |      0MiB / 15360MiB |      0%      Default |
   |                               |                      |                  N/A |
   +-------------------------------+----------------------+----------------------+
```

### Using JupyterLab

After your server fires up, you will be dropped into a JupyterLab environment.

If you are new to JupyterLab, you might want to peruse the [user guide](https://jupyterlab.readthedocs.io/en/stable/user/interface.html).

### Shutting Down Your Server

Your server will shut down automatically after a period of inactivity.
However, if you know you are done working, it's best to shut it down directly.
To shut it down, go to <https://leap.2i2c.cloud/hub/home> and click the big red button that says "Stop My Server"

<img width="800" alt="image" src="https://user-images.githubusercontent.com/1197350/167768526-7742a260-d353-4bdb-b9d0-36e9cc17aba1.png">

You can also navigate to this page from JupyterLab by clicking the `File` menu and going to `Hub Control Panel`.
