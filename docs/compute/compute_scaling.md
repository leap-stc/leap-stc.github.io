# Choosing a Server Image and Resources

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

Use this for training deep learning models or doing heavy inference.

- **NVIDIA Tesla T4**, 24GB RAM, 8 CPUs
- Compatible with all images
- Greatly accelerates TensorFlow and PyTorch training

#### Why not use GPU by default?

While GPU can accelerate certain workloads, it's not always the best choice for the following reasons:

- **Most tasks don't benefit:** Plotting, pandas/xarray analysis, or basic modeling run just as fast (or faster) on CPUs.
- **Shared, limited resources:** GPUs are a shared resource across LEAP. Using them when not needed can block others who rely on them for large-scale work.
- **More costly:** GPUs cost significantly more than CPU resources.

!!! note

    Use GPU only if your workflow truly needs it.

### Devs Only

The **Devs Only** option provides access to a full compute node (**~512 GP RAM, ~64 CPUs**). Only some users will see the **Devs Only** configuration. Access to this option is typically:

- Granted manually by administrators
- Enabled only for users with specific roles (e.g., image developers, system testers, researchers doing large-scale performance testing)

You typically won't need this unless you have been instructed to use it. It should be reserved for compute-intensive or infrastrcuture-critical tasks.

## How to choose:

Here is a simplified guide on how to choose the appropriate image and compute configuration:;

| Scenario                                            | Recommended Setup                                     |
| --------------------------------------------------- | ----------------------------------------------------- |
| Editing a notebook, small CSVs                      | Base Pangeo Notebook + 8 GB CPU                       |
| Plotting large NetCDF files with xarray             | Base Pangeo Notebook + 16–64 GB CPU                   |
| Visualizing high-resolution model outputs           | Base Pangeo Notebook + 16–64 GB CPU                   |
| Preprocessing large climate or satellite datasets   | Base Pangeo Notebook + 64–128 GB CPU                  |
| Large parallel Dask workloads                       | Base Pangeo Notebook + 32–128 GB CPU                  |
| Interactive Dask dashboard or distributed workflows | Base Pangeo Notebook + 32–128 GB CPU                  |
| Running large batch inference                       | PyTorch/TensorFlow ML Notebook + **GPU**              |
| Debugging or inference on smaller models            | PyTorch/TensorFlow ML Notebook + CPU                  |
| Training a PyTorch model                            | PyTorch ML Notebook + **GPU**                         |
| Running a TensorFlow model at scale                 | TensorFlow ML Notebook + **GPU**                      |
| Fine-tuning pre-trained deep learning models        | PyTorch/TensorFlow ML Notebook + **GPU**              |
| Hyperparameter tuning / grid search (ML)            | PyTorch/TensorFlow ML Notebook + **GPU** or Devs Only |
| Generating synthetic datasets or simulations        | Base Pangeo Notebook + 16–128 GB CPU                  |
| Testing new images or infrastructure                | Devs Only + any image                                 |

!!! tip

    If you are not sure which one to pick, then start with the Base Pangeo Notebook + 8-16 GB CPU. You can always stop your server and restart with a different configuration
