# Data Overview

LEAP makes use of LOTS of data.

This section explains how data is formatted, stored, transferred, and ultimately shared within LEAP.

- [File Formats][file-formats] describe which formats to use (Zarr, NetCDF, etc.) for cloud-ready, reproducible workflows.
- [Data Locations][where-data-lives] discusses the different areas data can be stored depending on the stage of work.
- The [Data Lifecycle][data-lifecycle] shows the big picture: ingestion, management, and cleanup.
- [Data Tools][data-tools] explains the tools for moving data between systems, including `fsspec/gcsfs` and `rclone`.
- The [Data Catalog][leap-data-catalog] describes how to discover datasets once they’ve been published with traceable provenance.
