# GeoNadir FAIR Data Dataset Structure and Content

Status: Draft for AWS Open Data Sponsorship Program application

This document describes the planned public structure for **GeoNadir FAIR Data**,
a proposed open-data collection of UAV survey datasets published through
AWS-hosted cloud-native geospatial files and a STAC-compatible catalog.

The intended collection ID is:

```text
geonadir-fair-data
```

This document is forward-looking. It describes the public dataset structure
GeoNadir intends to publish for open-data users; it is not a direct mirror of
GeoNadir's internal workspace, project, or permission model.

## Overview

GeoNadir FAIR Data will provide openly licensed UAV survey datasets contributed
through GeoNadir and structured for discovery, cloud-native access, and
geospatial analysis.

Each public dataset will represent one UAV survey. The required files are a
normalized metadata file and a ZIP package of the original raw images. Where
available, a dataset may also include an RGB orthomosaic, multispectral
orthomosaic, digital surface model, or digital terrain model.

Users will discover datasets through a STAC-compatible catalog and access the
files directly from AWS object storage. The catalog will allow users to find
datasets by capture date, upload or publication date, location, and selected
metadata fields such as IUCN habitat classification.

## Public Dataset Scope

The AWS Open Data collection will include only datasets that GeoNadir is able
to publish publicly under an open, no-fee license.

Private workspace data, authenticated user data, unpublished datasets, and
datasets without sufficient permission for open publication are out of scope
for this public collection.

If GeoNadir later exposes authenticated STAC access for private datasets, that
will be handled as a separate catalog or separate collection. The AWS Open Data
collection should remain public-only.

## Access Pattern

GeoNadir intends to expose the public dataset through a STAC-compatible access
layer.

Planned public entry points:

```text
https://data.geonadir.com/stac/
https://data.geonadir.com/stac/collections/geonadir-fair-data
https://data.geonadir.com/stac/collections/geonadir-fair-data/items
https://data.geonadir.com/stac/search
```

The STAC catalog is the discovery layer. The S3 bucket is the file storage
layer. Users should not need to understand the internal bucket layout to search
for datasets.

### Two access paths (STAC is optional)

The files are addressable directly on S3 through the stable
`datasets/{item_id}/` key convention described below. This means the data is
fully usable with plain AWS tooling (`boto3`, `rasterio`, GDAL, QGIS) **without
STAC** — list the `datasets/` prefix, read a survey's `metadata.json`, and open
its COGs by their known keys.

STAC is a discovery **enhancement** layered on top of that S3 layout, providing
search by date, area of interest, and habitat. GeoNadir may bring the STAC
endpoint online **after** the data itself is published, so tutorials and
integrations should not assume STAC is available at launch. The repository ships
two tutorial notebooks accordingly: a basic-AWS/S3 notebook that runs against
the bucket alone, and a STAC notebook for when the catalog is live.

## Planned S3 Layout

The planned S3 layout is deliberately simple. Dataset discovery will happen
through STAC, so the S3 prefixes do not need to encode country, region, year, or
other search facets.

```text
s3://geonadir-fair-data/
  stac/
    catalog.json
    collections/
      geonadir-fair-data/
        collection.json
        items/
          {item_id}.json

  datasets/
    {item_id}/                          # item_id = {dataset_id}-{uav_uid}
      {dataset_id}-metadata.json
      {dataset_id}-raw_images.zip
      {dataset_id}-rgb_ortho.tif
      {dataset_id}-multispec_ortho.tif
      {dataset_id}-dsm.tif
      {dataset_id}-dtm.tif
```

Only files that exist for a dataset will be listed as STAC assets. For example,
a dataset without a DTM will not include a `dtm` asset.

### File naming convention

Every object inside a dataset folder follows the convention
`{dataset_id}-{file_type}.{ext}`. The folder is the item id,
`{dataset_id}-{uav_uid}`; the file prefix is the numeric `{dataset_id}` alone
(not the full item id).

| Asset key | File name |
| --- | --- |
| `metadata` | `{dataset_id}-metadata.json` |
| `raw_images` | `{dataset_id}-raw_images.zip` |
| `ortho` | `{dataset_id}-rgb_ortho.tif` |
| `multispec_ortho` | `{dataset_id}-multispec_ortho.tif` |
| `dsm` | `{dataset_id}-dsm.tif` |
| `dtm` | `{dataset_id}-dtm.tif` |

The `{item_id}` is a stable public identifier derived from the dataset's id and
UUID, and remains stable even if other details change.

## File Formats

| Asset | Format | Required | Description |
| --- | --- | --- | --- |
| `{dataset_id}-metadata.json` | JSON | Yes | Normalized dataset metadata (the STAC Item) |
| `{dataset_id}-raw_images.zip` | ZIP of JPEG images | Yes | Original UAV images |
| `{dataset_id}-rgb_ortho.tif` | Cloud Optimized GeoTIFF | Optional | RGB orthomosaic generated from UAV imagery |
| `{dataset_id}-multispec_ortho.tif` | Cloud Optimized GeoTIFF | Optional | Multispectral orthomosaic, where available |
| `{dataset_id}-dsm.tif` | Cloud Optimized GeoTIFF | Optional | Digital Surface Model |
| `{dataset_id}-dtm.tif` | Cloud Optimized GeoTIFF | Optional | Digital Terrain Model |

Cloud Optimized GeoTIFFs can be read directly by tools such as GDAL, Rasterio,
QGIS, and cloud-native geospatial workflows without downloading the full file.

## STAC Model

GeoNadir FAIR Data will use the following STAC model:

```text
Catalog
  Collection: geonadir-fair-data
    Item: one public GeoNadir dataset
      Assets: metadata, raw images, and optional processed products
```

One public GeoNadir dataset maps to one STAC Item. Each dataset's
`metadata.json` **is** that STAC Item. A GeoNadir workspace or project can
contain multiple datasets and internal application state; those internal
concepts are not part of the public STAC contract.

## Collection Definition

| Field | Planned Value |
| --- | --- |
| `id` | `geonadir-fair-data` |
| `type` | `Collection` |
| `title` | `GeoNadir FAIR Data` |
| `description` | Public UAV survey datasets and related products contributed through GeoNadir |
| `license` | `CC-BY-4.0` |
| `extent.spatial` | Overall spatial extent of all published items |
| `extent.temporal` | Overall capture-date range of all published items |
| `summaries` | Collection-wide value summaries for selected fields |
| `item_assets` | Definitions of the asset keys that may appear on items |
| `stac_extensions` | STAC extensions used by the collection or its item assets |

## License and Attribution

The public collection uses:

```text
CC-BY-4.0
```

Attribution is represented with the STAC-standard `providers` array, so generic
STAC tooling reads it directly.

### Attribution (`providers`)

Each item lists the parties involved in producing and publishing the data, using
standard STAC provider roles:

```json
"providers": [
  { "name": "Example field team", "roles": ["producer"] },
  { "name": "Example Workspace", "roles": ["licensor"] },
  { "name": "GeoNadir", "roles": ["processor", "host"], "url": "https://geonadir.com" }
]
```

- **producer** — who captured the data (uploader-supplied, when provided).
- **licensor** — the contributing organisation or dataset owner.
- **processor / host** — GeoNadir, which processed and published the dataset.

When the same party is both producer and licensor (a solo contributor who
captured their own data), they appear once with both roles. Producer and
licensor values are uploader-supplied and should be treated as attribution
metadata rather than verified authority records.

## Item Structure

Each STAC Item represents one public GeoNadir dataset.

| Field | Description |
| --- | --- |
| `type` | `Feature` |
| `stac_version` | `1.1.0` |
| `stac_extensions` | EO, File, Alternate Assets |
| `id` | Stable public item identifier, `{dataset_id}-{uav_uid}` |
| `bbox` | Bounding box in WGS84 longitude/latitude |
| `geometry` | GeoJSON geometry in WGS84 (survey footprint rectangle) |
| `properties` | Searchable and descriptive item metadata |
| `assets` | Downloadable or streamable files |
| `links` | `self` (the item's own URL) and `via` (the GeoNadir dataset page) |

When the STAC catalog is online it adds the `collection` and `root` links that
tie each item into the catalog; the standalone `metadata.json` carries only the
`self` and `via` links.

### Geometry and Bbox

In STAC, `geometry` and `bbox` are different fields.

`bbox` is the rectangular bounding box:

```json
[west, south, east, north]
```

`geometry` is the GeoJSON geometry used for spatial search. GeoNadir publishes a
rectangular polygon covering the survey footprint, in WGS84 longitude/latitude,
wound counter-clockwise per the GeoJSON right-hand rule.

## Item Properties

| Field | Type | Description |
| --- | --- | --- |
| `title` | string | Human-readable dataset title |
| `description` | string | Public dataset description |
| `datetime` | datetime | STAC-standard capture datetime used for temporal search |
| `created` | datetime | Upload or first publication datetime |
| `updated` | datetime | Last metadata or asset modification datetime |
| `license` | string | Item license (`CC-BY-4.0`) |
| `gsd` | number | Ground sampling distance in metres |
| `instruments` | array | Camera/sensor model, e.g. `["M3M"]`, where available |
| `platform` | string | Aircraft, e.g. `"DJI M3M"`, where available |
| `providers` | array | Attribution — see [Attribution](#attribution-providers) |
| `geonadir:dataset_id` | integer | Numeric dataset identifier |
| `geonadir:raw_image_count` | integer | Number of source images represented by the dataset |
| `geonadir:area_m2` | number | Survey area in square metres |
| `geonadir:relative_altitude_m` | number | Relative flight altitude in metres, where available |
| `geonadir:sensor_metadata_source` | string | Source of sensor metadata, e.g. `first_image_exif` |
| `geonadir:location_name` | string | Human-readable location name, where available |
| `geonadir:tags` | array | Uploader-supplied free-text tags |
| `geonadir:iucn_habitat` | array | IUCN habitat classification(s) applied to the dataset |

Sensor fields (`instruments`, `platform`, `relative_altitude_m`) come from the
imagery metadata and are published only when they can be read reliably; they are
omitted otherwise. `platform` is a vendor+model string such as `"DJI M3M"`.

## Band Metadata

Band metadata follows the STAC 1.1 `bands` construct and is read from each
GeoTIFF's own band table, so it always reflects the real file. Every band has:

- **`name`** — `B1`, `B2`, … in physical band order.
- **`description`** — the band's human-readable name from the GeoTIFF (e.g.
  `Red`, `NIR`, `Rededge705`, `Alpha`).
- **`eo:common_name`** — a standard EO name (`red`, `green`, `blue`, `nir`,
  `rededge`, `coastal`, …), included **only** when the band maps to one. Narrow
  bands such as `Blue444` or `Rededge705` have no standard common name, so they
  carry a `description` only.

For an RGB orthomosaic (with an alpha band):

```json
"bands": [
  { "name": "B1", "description": "Red",   "eo:common_name": "red" },
  { "name": "B2", "description": "Green", "eo:common_name": "green" },
  { "name": "B3", "description": "Blue",  "eo:common_name": "blue" },
  { "name": "B4", "description": "Alpha" }
]
```

For a multispectral orthomosaic the band array follows the physical band order —
for example a DJI Mavic 3 Multispectral carrying red, green, near-infrared, and
red-edge bands plus an alpha band:

```json
"bands": [
  { "name": "B1", "description": "Red",     "eo:common_name": "red" },
  { "name": "B2", "description": "Green",   "eo:common_name": "green" },
  { "name": "B3", "description": "NIR",     "eo:common_name": "nir" },
  { "name": "B4", "description": "RedEdge", "eo:common_name": "rededge" },
  { "name": "B5", "description": "Alpha" }
]
```

The exact band set and order vary by sensor; always read `bands` from the asset
metadata rather than assuming a fixed layout.

## Item Assets

STAC assets are the downloadable or streamable files attached to an item.

| Asset Key | Required | Type | Roles | Description |
| --- | --- | --- | --- | --- |
| `raw_images` | Yes | `application/zip` | `source` | Original UAV images as a ZIP package |
| `ortho` | Optional | `image/tiff; application=geotiff; profile=cloud-optimized` | `data`, `visual` | RGB orthomosaic COG |
| `multispec_ortho` | Optional | `image/tiff; application=geotiff; profile=cloud-optimized` | `data` | Multispectral orthomosaic COG |
| `dsm` | Optional | `image/tiff; application=geotiff; profile=cloud-optimized` | `data`, `elevation` | Digital Surface Model |
| `dtm` | Optional | `image/tiff; application=geotiff; profile=cloud-optimized` | `data`, `elevation` | Digital Terrain Model |

The item's own `metadata.json` is reachable through the `self` link rather than
as an asset. Example asset definition:

```json
"assets": {
  "raw_images": {
    "href": "https://geonadir-fair-data.s3.ap-southeast-2.amazonaws.com/datasets/{item_id}/{dataset_id}-raw_images.zip",
    "type": "application/zip",
    "roles": ["source"],
    "title": "Original raw images",
    "file:size": 5368709120,
    "alternate": {
      "s3": {
        "href": "s3://geonadir-fair-data/datasets/{item_id}/{dataset_id}-raw_images.zip",
        "alternate:name": "S3"
      }
    }
  },
  "ortho": {
    "href": "https://geonadir-fair-data.s3.ap-southeast-2.amazonaws.com/datasets/{item_id}/{dataset_id}-rgb_ortho.tif",
    "type": "image/tiff; application=geotiff; profile=cloud-optimized",
    "roles": ["data", "visual"],
    "title": "RGB orthomosaic",
    "bands": [
      { "name": "B1", "description": "Red", "eo:common_name": "red" },
      { "name": "B2", "description": "Green", "eo:common_name": "green" },
      { "name": "B3", "description": "Blue", "eo:common_name": "blue" },
      { "name": "B4", "description": "Alpha" }
    ],
    "file:size": 734003200,
    "alternate": {
      "s3": {
        "href": "s3://geonadir-fair-data/datasets/{item_id}/{dataset_id}-rgb_ortho.tif",
        "alternate:name": "S3"
      }
    }
  }
}
```

A complete reference item is committed at
[`datasets/geonadir-fair-data/metadata.template.json`](datasets/geonadir-fair-data/metadata.template.json).

### Asset hrefs: `https://` primary, `s3://` alternate

Asset `href` values use `https://` as the primary access method — what a browser, a
notebook, and most STAC tooling expect, and the more accessible choice for a FAIR
catalogue. The `s3://` form (which the AWS CLI and GDAL `/vsis3/` prefer) is carried
alongside it through the STAC **alternate-assets** extension, so both access methods
are published without forcing a choice. Both URLs are anonymously readable on the
public-read bucket, and the COGs serve HTTP range requests over `https://` as well as
`s3://`, so a client can stream a window or a single overview by either path.

## Collection Summaries

Collection summaries describe expected values or ranges across all items in the
collection. They help users understand what fields exist without inspecting
every item.

Planned summaries for the first public version:

```json
"summaries": {
  "license": ["CC-BY-4.0"],
  "gsd": { "minimum": 0.0, "maximum": 1.0 },
  "geonadir:area_m2": { "minimum": 0.0, "maximum": 100000000.0 },
  "geonadir:relative_altitude_m": { "minimum": 0.0, "maximum": 1000.0 },
  "geonadir:iucn_habitat": [
    "Forest", "Savanna", "Shrubland", "Grassland", "Wetlands", "Rocky",
    "Desert", "Neritic", "Oceanic", "Deep ocean floor", "Intertidal",
    "Artificial - terrestrial", "Other"
  ]
}
```

The numeric range values above are illustrative. The final `minimum` and
`maximum` values will be calculated from the first public publication set.

## STAC Extensions

GeoNadir FAIR Data uses STAC extensions only where they add useful, standardized
meaning.

| Extension | Why it is useful |
| --- | --- |
| EO | Describes optical bands such as red, green, blue, red edge, and NIR |
| File | Publishes file size for each asset |
| Alternate Assets | Carries both an `https://` (primary) and an `s3://` href for each asset without choosing one |

```json
"stac_extensions": [
  "https://stac-extensions.github.io/eo/v2.0.0/schema.json",
  "https://stac-extensions.github.io/file/v2.1.0/schema.json",
  "https://stac-extensions.github.io/alternate-assets/v1.2.0/schema.json"
]
```

Projection metadata is not duplicated in the item properties. Each GeoTIFF
contains its own coordinate reference system and geotransform, which users can
inspect directly with GDAL, Rasterio, or QGIS.

## Queryable Fields

The first public STAC API keeps queryables minimal and reliable.

| Queryable | Description |
| --- | --- |
| `datetime` | Capture datetime |
| `geonadir:iucn_habitat` | IUCN habitat classification |
| `geometry` | Spatial intersection using the item geometry |

Optional later queryables: `gsd`, `geonadir:area_m2`,
`geonadir:raw_image_count`, `platform`.

## Example Search Use Cases

Users should be able to ask questions such as:

- Find public UAV survey datasets captured within a date range.
- Find datasets uploaded after a specific date.
- Find datasets intersecting a bounding box or area of interest.
- Find datasets associated with a specific IUCN habitat classification.
- Load an orthomosaic COG directly into GDAL, Rasterio, QGIS, or another
  cloud-native geospatial tool when that asset is available.

Example conceptual STAC search request:

```json
{
  "collections": ["geonadir-fair-data"],
  "datetime": "2024-01-01T00:00:00Z/2024-12-31T23:59:59Z",
  "intersects": {
    "type": "Polygon",
    "coordinates": [
      [
        [151.0, -34.0],
        [151.2, -34.0],
        [151.2, -33.8],
        [151.0, -33.8],
        [151.0, -34.0]
      ]
    ]
  },
  "filter": "geonadir:iucn_habitat = 'Forest'"
}
```

## Example Item Skeleton

A complete, populated example is committed at
[`datasets/geonadir-fair-data/metadata.template.json`](datasets/geonadir-fair-data/metadata.template.json).
A condensed skeleton:

```json
{
  "type": "Feature",
  "stac_version": "1.1.0",
  "stac_extensions": [
    "https://stac-extensions.github.io/eo/v2.0.0/schema.json",
    "https://stac-extensions.github.io/file/v2.1.0/schema.json",
    "https://stac-extensions.github.io/alternate-assets/v1.2.0/schema.json"
  ],
  "id": "{item_id}",
  "bbox": [151.0, -34.0, 151.2, -33.8],
  "geometry": {
    "type": "Polygon",
    "coordinates": [[
      [151.0, -34.0], [151.2, -34.0], [151.2, -33.8], [151.0, -33.8], [151.0, -34.0]
    ]]
  },
  "properties": {
    "title": "Example GeoNadir UAV survey dataset",
    "description": "Public UAV survey dataset contributed through GeoNadir.",
    "datetime": "2025-04-18T02:14:00Z",
    "created": "2025-04-20T08:31:00Z",
    "updated": "2025-05-02T03:12:00Z",
    "license": "CC-BY-4.0",
    "gsd": 0.026,
    "instruments": ["M3M"],
    "platform": "DJI M3M",
    "providers": [
      { "name": "Example field team", "roles": ["producer"] },
      { "name": "Example Workspace", "roles": ["licensor"] },
      { "name": "GeoNadir", "roles": ["processor", "host"], "url": "https://geonadir.com" }
    ],
    "geonadir:dataset_id": 12760,
    "geonadir:raw_image_count": 420,
    "geonadir:area_m2": 124300.0,
    "geonadir:relative_altitude_m": 90.5,
    "geonadir:sensor_metadata_source": "first_image_exif",
    "geonadir:location_name": "Example Bay, Australia",
    "geonadir:tags": ["coastal", "mangrove"],
    "geonadir:iucn_habitat": ["Forest"]
  },
  "links": [
    { "rel": "self", "type": "application/json", "href": "https://.../{dataset_id}-metadata.json" },
    { "rel": "via", "type": "text/html", "href": "https://data.geonadir.com/image-collection-details/{dataset_id}" }
  ],
  "assets": {
    "ortho": { "...": "RGB orthomosaic COG with bands + file:size + s3 alternate" },
    "raw_images": { "...": "application/zip with file:size + s3 alternate" }
  }
}
```

## Planned Tutorial Notebook

GeoNadir expects to provide Jupyter notebooks with real examples after the
initial application stage. The notebooks demonstrate how to:

- Open the GeoNadir FAIR Data STAC catalog (or the S3 bucket directly).
- Search by capture date and area of interest.
- Filter by IUCN habitat classification.
- Inspect item metadata and asset links.
- Open an orthomosaic COG directly from S3 when that asset is available.
- Read raster metadata such as CRS, transform, resolution, and band count.
- Display the orthomosaic and compute simple area or pixel summaries.
