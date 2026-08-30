# GeoNadir FAIR Data — Open Data documentation & examples

This repository is the documentation and examples home for **GeoNadir FAIR Data**,
an open collection of UAV (drone) survey datasets contributed through GeoNadir
and published through the [AWS Open Data Sponsorship Program](https://aws.amazon.com/opendata/open-data-sponsorship-program/).

Datasets are published as cloud-native geospatial files (Cloud Optimized
GeoTIFFs, raw-image ZIPs, JSON metadata). The collection contains only data
GeoNadir can publish publicly under an open, no-fee license (CC-BY-4.0).

**Two access paths.** The files are addressable directly on Amazon S3 by a
simple key convention, so everything works with plain `boto3` + `rasterio`
from day one. A **STAC-compatible catalog** is layered on top for richer
search (by date, area, and habitat). The direct-S3 path does not depend on
STAC — STAC is an enhancement that may come online later than the data itself.

- **Browse the collection:** https://data.geonadir.com/fairgeo (live after launch)
- **Registry of Open Data landing page:** https://registry.opendata.aws/geonadir-fair-data (live after launch)
- **STAC catalog:** https://data.geonadir.com/stac/collections/geonadir-fair-data (live after STAC is set up)

## Contents

| Path | What it is |
| --- | --- |
| [`OPEN_DATA_STRUCTURE.md`](OPEN_DATA_STRUCTURE.md) | **Descriptive documentation** — dataset structure, S3 layout, STAC model, item/asset schema, band metadata, licensing. This is the URL referenced in the Registry `Documentation` field. |
| [`datasets/geonadir-fair-data.yaml`](datasets/geonadir-fair-data.yaml) | Draft **Registry of Open Data** entry (submitted as a draft PR to [awslabs/open-data-registry](https://github.com/awslabs/open-data-registry)). |
| [`datasets/geonadir-fair-data/get-to-know-a-dataset.ipynb`](datasets/geonadir-fair-data/get-to-know-a-dataset.ipynb) | **"Get To Know A Dataset" (basic AWS)** — the primary tutorial. Uses **only S3** (`boto3` + `rasterio`): list surveys, read metadata, open COGs directly, NDVI worked example, community challenge. Runs the moment the bucket is live, no STAC needed. |
| [`datasets/geonadir-fair-data/get-to-know-a-dataset-stac.ipynb`](datasets/geonadir-fair-data/get-to-know-a-dataset-stac.ipynb) | **"Get To Know A Dataset" (STAC)** — the same tour, but discovering surveys through the STAC catalog for richer search. For when the STAC endpoint is online. |

> **Draft status.** GeoNadir FAIR Data is mid-onboarding. Until the dedicated
> AWS Open Data account's S3 bucket is live, the `s3://` and web URLs above are
> provisional. In the notebooks, cells that need only the bucket are marked
> `[RUNS AT LAUNCH]`; cells that need the STAC endpoint are marked `[NEEDS STAC]`.
