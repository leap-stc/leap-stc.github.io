# Compute Scaling

When launching a server on the LEAP JupyterHub, you'll be asked to select a compute configuration. This guide helps you choose the right **image** and **hardware resources (RAM and CPU/GPU)** for your workflow.

## Image Types

Each image contains a different set of pre-installed software packages. Choose the image that fits your computing needs:

| Image Name                        | Use When You Need...                                                                                              |
| --------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| **Base Pangeo Notebook**          | General scientific stack (e.g., xarray, dask, matplotlib). Ideal for climate, ocean, and earth science workflows. |
| **Pangeo PyTorch ML Notebook**    | PyTorch for machine learning. Runs on CPU or GPU depending on your hardware choice.                               |
| **Pangeo TensorFlow ML Notebook** | TensorFlow for machine learning. Runs on CPU or GPU depending on your hardware choice.                            |
| **Other...**                      | Enter a custom image URL                                                                                          |

## Resource Options

Choose a CPU/GPU configuration based on the size of your data and the complexity of your tasks.

### CPU

Use this for data exploration, lightweight model runs, or debugging.

| Option                  | Use Case                                                 |
| ----------------------- | -------------------------------------------------------- |
| **~8 GB, ~1.0 CPU**     | Small notebooks, light plotting, CSVs or small NetCDF.   |
| **~16–64 GB, ~2–8 CPU** | Medium-sized xarray/dask workloads, ML prototyping.      |
| **~128 GB, ~16 CPU**    | Large simulations, ensemble runs, or parallel workflows. |

### GPU

Selecting the **GPU** profile starts your container on a node with a **single** GPU attached. Multi-GPU servers are not offered on the hub. If you need data-parallel training across several GPUs, see [Batch Jobs](https://leap-stc.github.io/compute/skypilot/).

Three GPU types are available:

| GPU type            | VRAM  | RAM (limit)    | CPUs | Google Cloud node | Who can select it                       |
| ------------------- | ----- | -------------- | ---- | ----------------- | --------------------------------------- |
| **NVIDIA Tesla T4** | 16 GB | ~30 GB (29 GB) | 8    | `n1-standard-8`   | Anyone with GPU access (default choice) |
| **NVIDIA L4**       | 24 GB | ~16 GB (15 GB) | 4    | `g2-standard-4`   | Members of `leap-stc:large-gpu`         |
| **NVIDIA A100**     | 40 GB | ~85 GB (82 GB) | 12   | `a2-highgpu-1g`   | Members of `leap-stc:large-gpu`         |

The first RAM figure matches the launcher dropdown (the node's total). The figure in parentheses is the hard limit on your server, so budget against that one.

!!! note "A bigger GPU isn't always faster"

    Start on the T4 and move up only when you hit a specific wall like running out of GPU memory. The L4 comes with a smaller machine around it, so choosing it just to get "more GPU" can leave you slower than where you started.

#### Data loading is often the bottleneck

Much LEAP work trains comparatively modest models on very large datasets read from cloud object storage. The GPU is frequently *not* the constraint: the CPUs feeding it are. LEAP's own [ClimSim](https://github.com/leap-stc/ClimSim) tooling carries this warning in its data utilities: used as a training dataloader, I/O can slow training down considerably.

Practical implications:

- **Profile before upgrading.** If `nvidia-smi` shows low GPU utilization during training, a bigger GPU will not help. More CPUs, larger chunk-aligned reads, or caching a subset to local disk will.
- **Watch the L4's 4 CPUs.** Multi-worker PyTorch `DataLoader` setups and Zarr decompression are CPU-bound. Four cores and a ~15 GB memory ceiling can leave a 24 GB GPU idle.
- **Read along your chunk boundaries.** Random access across a Zarr store's chunk grid amplifies reads and starves the GPU regardless of which one you picked.
- **Consider CPU-only for offline evaluation.** Scoring predictions, computing metrics, and plotting rarely benefit from a GPU.

#### Requesting access to the L4 and A100

Both large GPUs are gated behind a GitHub team so these scarce, expensive nodes stay available to the researchers who need them.

1. Ask the LEAP Data and Compute Team (see [Contact](https://leap-stc.github.io/support/contact/)) to add your GitHub username to **`leap-stc:large-gpu`**. Include a short description of your workflow and why the T4 is not sufficient.
1. Accept the GitHub team invitation.
1. **Log out of the hub and log back in.** Your team membership is read at login, so the new options will not appear until you re-authenticate.

#### Availability and startup time

All GPU node pools run in **us-central1** and scale down to zero when unused.

- **Expect a slow first launch.** If no GPU node of your chosen type is already running, one must be provisioned before your server starts. This commonly takes several minutes.
- **Capacity is not guaranteed.** Google Cloud GPU availability varies by zone and week. The **L4 has been persistently scarce**; the A100 is generally easier to obtain.
- **If your server never starts**, try a different GPU type, or wait and retry later. If it fails for more than a few hours, [contact us](https://leap-stc.github.io/support/contact/).

#### Checking that your GPU is working

From a terminal on your server:

```bash
nvidia-smi
```

Or in a notebook:

```python
import torch

print(torch.cuda.is_available())
print(torch.cuda.get_device_name(0))
```

```python
import tensorflow as tf

print(tf.config.list_physical_devices("GPU"))
```

`nvidia-smi` is also the quickest way to check GPU *utilization* and VRAM use while training.

!!! warning "Custom images need CUDA-enabled builds"

    The GPU profile's image dropdown offers the PyTorch and TensorFlow ML notebooks, which are built for GPU use. If you supply your own image via the free-form image option, it must contain CUDA-enabled builds of your framework, otherwise the GPU is attached but never used, and we are billed for it anyway.

#### Why not use GPU by default?

While GPU can accelerate certain workloads, it's not always the best choice:

- **Most tasks don't benefit:** Plotting, pandas/xarray analysis, and basic modeling run just as fast (or faster) on CPUs.
- **Shared, limited resources:** GPUs are a shared resource across LEAP. Using them when not needed can block others who rely on them for large-scale work.
- **More costly:** GPUs cost significantly more than CPU resources, and the A100 is the most expensive option on the hub by a wide margin.

!!! note

    Please use GPUs only if your workflow truly needs it, and **stop your server as soon as you are done**, an idle GPU server bills at the full GPU rate.

!!! tip "Running a workshop or class that needs GPUs?"

    A large GPU can be time-sliced across users. Time-slicing interleaves work rather than partitioning it: everyone shares one pool of VRAM with no memory or fault isolation, so one participant can OOM the others. Good for teaching, poor for unattended training. Not enabled by default.

## How to choose:

Here is a simplified guide on how to choose the appropriate image and compute configuration:

| Scenario                                                   | Image                | Hardware                                          |
| ---------------------------------------------------------- | -------------------- | ------------------------------------------------- |
| Editing a notebook, small CSVs                             | Base Pangeo          | ~8 GB CPU                                         |
| Plotting large NetCDF, visualizing model output            | Base Pangeo          | 16–64 GB CPU                                      |
| Preprocessing large climate/satellite datasets             | Base Pangeo          | 64–128 GB CPU                                     |
| Dask workloads, distributed workflows                      | Base Pangeo          | 32–128 GB CPU                                     |
| Generating synthetic datasets or simulations               | Base Pangeo          | 16–128 GB CPU                                     |
| Inference on small models                                  | PyTorch / TensorFlow | CPU                                               |
| Debugging a training loop before a full run                | PyTorch / TensorFlow | **T4**                                            |
| MLP or small CNN parameterizations, moderate batches       | PyTorch / TensorFlow | **T4**                                            |
| Training a PyTorch or TensorFlow model                     | PyTorch / TensorFlow | **T4** to start                                   |
| Code that needs bf16 or FP8                                | PyTorch / TensorFlow | **L4**                                            |
| Training that runs out of GPU memory on the T4             | PyTorch / TensorFlow | **L4**, then **A100** if 24 GB is also short      |
| High-throughput batch inference                            | PyTorch / TensorFlow | **L4**; T4 as a cheaper fallback                  |
| Large or memory-hungry training; transformers; big batches | PyTorch / TensorFlow | **A100**                                          |
| Fine-tuning pre-trained models                             | PyTorch / TensorFlow | **A100**; L4 only for small vision models or LoRA |
| Heavy Zarr/cloud dataloading during training               | PyTorch / TensorFlow | **T4**; fix the I/O first                         |
| Hyperparameter tuning / grid search                        | PyTorch / TensorFlow | Any GPU, or Devs Only                             |

!!! tip

    If you are not sure which one to pick, then start with the Base Pangeo Notebook + 8-16 GB CPU. You can always stop your server and restart with a different configuration.
