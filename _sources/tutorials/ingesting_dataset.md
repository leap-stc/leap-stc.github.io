(tutorials.data.ingestion)=

# Data Ingestion Tutorial

New to cloud data ingestion? You're not alone! Transferring large volumes of climate data is not an easy process, and can feel like a tall order. Never fear! This tutorial is aimed completely at beginners, and is designed to provide the **what, why, and how** for data ingestion! "Data ingestion" in our usage means a programmatic way to download and transform data into [Analysis-Ready Cloud-Optimized (ARCO)](reference.arco) formats in a reproducible way, so that the dataset is available for the LEAP community and beyond (see [](reference.infrastructure.buckets) for who can access which resource).

## Requirements and Desiderata

We have additional requirements for the data ingestion to make the process sustainable and scalable:

- Process needs to be reproducible, e.g. when we want to reingest data to a different storage location
- Separation of concerns: The person who knows the dataset (the 'data expert') is in the unique position to encode their knowledge about the dataset into the recipe, but they should not be concerned with the details of how to execute it and where the data is ultimate stored. This is the responsibility of the Data and Compute team.

For clearer organization each dataset should reside in its own repository under the `leap-stc` github organization. Each of these repositories will be called a 'feedstock', which contains additional metadata files.

## How to get new data ingested

To start ingesting a dataset follow these steps:

1. Let the LEAP community and the Data and Computation Team know about this new dataset. We gather all ingestion requests in our ['leap-stc/data_management' issue tracker](https://github.com/leap-stc/data-management/issues). You should check existing issues with the tag ['dataset'](https://github.com/leap-stc/data-management/issues?q=is%3Aissue+is%3Aopen+label%3Adataset) to see if somebody else might have already requested this particular dataset. If that is not the case you can add a new [dataset_request](https://github.com/leap-stc/data-management/issues/new?assignees=&labels=dataset&projects=&template=new_dataset.yaml&title=New+Dataset+%5BDataset+Name%5D). Making these request in a central location enables others to see which datasets are currently being ingested and what the status is.
1. Use our [feedstock template](https://github.com/leap-stc/LEAP_template_feedstock) to create a feedstock repostory by following instructions in the README to get you started with either one of the above.
1. If issues arise please reach out to the [](support.data_compute_team)

```{note}
This does currently not provide a solution to handle datasets that have been produced by you (as e.g. part of a publication). We are working on formalizing a workflow for this type of data. Please reach out to the [](support.data_compute_team) if you have data that you would like to publish. See [](explanation.data-policy.types) for more.
```
