(explanation.data-policy)=

---
abbreviations:
  ARCO: Analysis-Ready Cloud-Optimized
---

(explanation.data-policy.types)=

## Types of Data Used at LEAP

Within the LEAP project we distinguish between several different types of data mostly based on whether the data was used or produced at LEAP and if the data is already accessible in {abbr}`ARCO` formats in the cloud.

```{admonition} LEAP produced
---
class: dropdown
---
Data that has been created or modified by LEAP researchers. We currently do not provide any way of ensuring that data is archived, and users should never rely on LEAP-Pangeo resources as the only replicate of valuable data (see also [](guides.data.ingestion)).
```

```{admonition} LEAP ingested
---
class: dropdown
---
Data that is already publically available but has been ingested into cloud storage in {abbr}`ARCO` formats. The actual data/metadata has not been modified from the original.
```

```{admonition} LEAP curated
---
class: dropdown
---
Data that is already available in {abbr}`ARCO` formats in publically accessible object storage. Adding this data to the LEAP-Pangeo Catalog enables us to visualize it with the Data Viewer, and collect all datasets of importance in one single location, but none of the data itself is modified.
```
