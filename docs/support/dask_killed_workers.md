# Dask "Killed Workers"

The "Killed Worker" message in dask can result due to a variety of reasons. Here are some of the common reasons why you might see such an error message

## Datasets Chunks too large

**Issue**

The default dask worker configuration can deal well with dataset chunk sizes of ~100MB. If the chunks of your data are significantly larger, your worker might crash just upon loading a few of these chunks into memory.

**Solution**

You can change the configuration of your dask workers and increase the memory each worker has to deal with larger chunks. You can adjust the memory by passing additional [options](https://gateway.dask.org/cluster-options.html) to the dask-gatway cluster upon creation:

```python
from dask_gateway import Gateway

gateway = Gateway()
options = gateway.cluster_options()

options.worker_memory = 10  # 10 GB of memory per worker.

# Create a cluster with those options
cluster = gateway.new_cluster(options)
cluster.scale(...)
client = cluster.get_client()
```

<!-- TODO: Add example how to change this in HTML repr -->

[Example with Solution](https://notebooksharing.space/view/2b6753a5ffe8ddfae1da3b8e2b5507e617de47eb25f758a20c92b62e7e650fd7#displayOptions=)
