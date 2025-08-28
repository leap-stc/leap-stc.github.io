## Dask / Xarray tips and tricks

The Xarray docs have a page on [Xarray + dask best practices](https://docs.xarray.dev/en/stable/user-guide/dask.html#best-practices).

#### Correct chunk sizes

When working with Xarray and Zarr, you should aim for data chunk sizes around **100MB**. Chunk sizes too small can overwhelm the Dask scheduler, while chunk sizes being to large can give you memory issues.

You can examine the chunk size and shape by viewing the html repr of an Xarray dataset in a Jupyter Notebook. If you click on the right-most database logo you should get a drop-down menu that shows the chunking information.

![Chunking](../assets/xarray_repr.png)

#### Delay computation until write

Xarray is great at lazy-computation, this means that is usually possible to run multiple operations before any computation is done. Generally it is a good idea to keep everything lazy (not eager!) until your finish your processing and write your data. For example, a `to_zarr()` call would trigger the computation. This can generally be accomplished by not calling `load()`, `.persist()` or `.compute()`.

#### Use the Dask dashboard

If you are using Dask distributed scheduler you can view the [dask dashboard](https://docs.dask.org/en/latest/dashboard.html) in a browser. This allows you to see memory usage, task progression and a bunch of other metrics all live.

```python
from distributed import Client

client = Client()
client
```
