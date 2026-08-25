# GeoNadir Fair Data Dataset Structure and Content

Status: Draft for AWS Open Data Sponsorship Program application

This document describes the planned public structure for **GeoNadir Fair Data**,
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

GeoNadir Fair Data will provide openly licensed UAV survey datasets contributed
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
    {item_id}/
      metadata.json
      raw-images.zip
      orthomosaic.tif
      dsm.tif
      dtm.tif
      multispectral-orthomosaic.tif
```

Only files that exist for a dataset will be listed as STAC assets. For example,
a dataset without a DTM will not include a `dtm` asset.

The `{item_id}` should be a stable public identifier. It may be derived from
GeoNadir's internal dataset ID and UUID, but the public identifier should remain
stable even if internal application details change.

## File Formats

| Asset | Format | Required | Description |
| --- | --- | --- | --- |
| `metadata.json` | JSON | Yes | Normalized dataset metadata and source metadata summary |
| `raw-images.zip` | ZIP of JPEG images | Yes | Original UAV images |
| `orthomosaic.tif` | Cloud Optimized GeoTIFF | Optional | RGB orthomosaic generated from UAV imagery |
| `dsm.tif` | Cloud Optimized GeoTIFF | Optional | Digital Surface Model |
| `dtm.tif` | Cloud Optimized GeoTIFF | Optional | Digital Terrain Model |
| `multispectral-orthomosaic.tif` | Cloud Optimized GeoTIFF | Optional | Multispectral orthomosaic, where available |

Cloud Optimized GeoTIFFs can be read directly by tools such as GDAL, Rasterio,
QGIS, and cloud-native geospatial workflows without downloading the full file.

## STAC Model

GeoNadir Fair Data will use the following STAC model:

```text
Catalog
  Collection: geonadir-fair-data
    Item: one public GeoNadir dataset
      Assets: metadata, raw images, and optional processed products
```

One public GeoNadir dataset maps to one STAC Item.

A GeoNadir workspace or project can contain multiple datasets, annotations,
derived rasters, styling choices, and collaboration state. Those internal
application concepts are not part of the public STAC contract. The public STAC
Item describes the published geospatial dataset and its downloadable assets.

## Collection Definition

The collection ID will be:

```text
geonadir-fair-data
```

The collection title will be:

```text
GeoNadir Fair Data
```

The collection will describe open UAV survey datasets contributed through
GeoNadir.

Planned collection-level fields:

| Field | Planned Value |
| --- | --- |
| `id` | `geonadir-fair-data` |
| `type` | `Collection` |
| `title` | `GeoNadir Fair Data` |
| `description` | Public UAV survey datasets and related products contributed through GeoNadir |
| `license` | `CC-BY-4.0` |
| `extent.spatial` | Overall spatial extent of all published items |
| `extent.temporal` | Overall capture-date range of all published items |
| `summaries` | Collection-wide value summaries for selected fields |
| `item_assets` | Definitions of the asset keys that may appear on items |
| `stac_extensions` | STAC extensions used by the collection or its item assets |

## License and Attribution

The public collection will use:

```text
CC-BY-4.0
```

Attribution will be represented with plain GeoNadir item properties. This
matches the metadata GeoNadir can populate reliably today.

## Attribution Metadata

GeoNadir stores attribution as simple dataset-level values. These values do not
try to classify contributors into a role vocabulary that GeoNadir does not
currently store.

Planned attribution fields:

| Field | Type | Description |
| --- | --- | --- |
| `geonadir:provided_by` | string | Public display value for the workspace owner associated with the dataset |
| `geonadir:captured_by` | string | Free-text value supplied by the uploader for who captured the data |
| `geonadir:institution` | string | Free-text institution value; defaults from the workspace name but can be edited |

The `provided_by` value will use the workspace owner's name. If that user has
not set up a name, GeoNadir will fall back to the username. The `captured_by`
and `institution` values are uploader-supplied strings and should be treated as
attribution metadata rather than verified authority records.

## Item Structure

Each STAC Item represents one public GeoNadir dataset.

Top-level item fields:

| Field | Description |
| --- | --- |
| `type` | `Feature` |
| `stac_version` | STAC version used by the catalog |
| `id` | Stable public item identifier |
| `collection` | `geonadir-fair-data` |
| `bbox` | Bounding box in WGS84 longitude/latitude |
| `geometry` | Bbox-derived GeoJSON geometry in WGS84 |
| `properties` | Searchable and descriptive item metadata |
| `assets` | Downloadable or streamable files |
| `links` | STAC links to the collection, root catalog, and item self URL |

### Geometry and Bbox

In STAC, `geometry` and `bbox` are different fields.

`bbox` is the rectangular bounding box:

```json
[west, south, east, north]
```

`geometry` is the GeoJSON geometry used for spatial search. For the initial
release, GeoNadir will use a rectangular polygon derived from the dataset
bounding box and record:

```json
"geonadir:geometry_source": "bbox"
```

## Item Properties

Planned item properties:

| Field | Type | Description |
| --- | --- | --- |
| `title` | string | Human-readable dataset title |
| `description` | string | Public dataset description |
| `datetime` | datetime | STAC-standard capture datetime used for temporal search |
| `created` | datetime | Upload or first publication datetime |
| `updated` | datetime | Last metadata or asset modification datetime |
| `license` | string | Item license, normally inherited from the collection |
| `gsd` | number | Ground sampling distance in meters |
| `eo:bands` | array | Band metadata when the item-level band set is simple and consistent |
| `geonadir:capture_datetime` | datetime | Human-readable alias of `datetime` |
| `geonadir:upload_datetime` | datetime | Human-readable alias of `created` |
| `geonadir:provided_by` | string | Public display value for the workspace owner associated with the dataset |
| `geonadir:captured_by` | string | Free-text value supplied by the uploader for who captured the data |
| `geonadir:institution` | string | Free-text institution value associated with the dataset |
| `geonadir:manufacturer` | string | Manufacturer when confidently extracted, such as `DJI` |
| `geonadir:camera_model` | string | Camera or sensor model from image metadata, where available |
| `geonadir:sensor_metadata_source` | string | Source of sensor metadata, such as `image_exif` |
| `geonadir:relative_altitude_m` | number | Relative flight altitude in metres, where available |
| `geonadir:area_m2` | number | Survey area in square metres |
| `geonadir:raw_image_count` | integer | Number of source images represented by the dataset |
| `geonadir:iucn_habitat` | string or array | IUCN habitat classification applied to the dataset |

GeoNadir will avoid publishing inferred drone model names unless the model is
available from a reliable source. For example, if only the manufacturer can be
confidently extracted, the item will publish `DJI` as the manufacturer rather
than inferring a specific model such as `DJI Phantom 4 Pro`.

## Band Metadata

Band metadata will follow common Sentinel and Landsat STAC conventions.

The band `name` will use uppercase local band identifiers such as:

```text
B1, B2, B3
```

The `common_name` will use standard EO common names such as:

```text
red, green, blue, rededge, nir
```

The order of the band array will match the band order in the GeoTIFF.

For an RGB orthomosaic:

```json
"eo:bands": [
  {
    "name": "B1",
    "common_name": "red",
    "description": "Red"
  },
  {
    "name": "B2",
    "common_name": "green",
    "description": "Green"
  },
  {
    "name": "B3",
    "common_name": "blue",
    "description": "Blue"
  },
  {
    "name": "B4",
    "description": "Alpha"
  }
]
```

For a multispectral orthomosaic, where the source metadata identifies the band
order:

```json
"eo:bands": [
  {
    "name": "B1",
    "common_name": "green",
    "description": "Green"
  },
  {
    "name": "B2",
    "common_name": "red",
    "description": "Red"
  },
  {
    "name": "B3",
    "common_name": "rededge",
    "description": "Red edge"
  },
  {
    "name": "B4",
    "common_name": "nir",
    "description": "Near infrared"
  }
]
```

## Item Assets

STAC assets are the downloadable or streamable files attached to an item.

Planned asset keys:

| Asset Key | Required | Type | Roles | Description |
| --- | --- | --- | --- | --- |
| `metadata` | Yes | `application/json` | `metadata` | Normalized item metadata |
| `raw_images` | Yes | `application/zip` | `source` | Original UAV images as a ZIP package |
| `orthomosaic` | Optional | `image/tiff; application=geotiff; profile=cloud-optimized` | `data` | RGB orthomosaic COG |
| `dsm` | Optional | `image/tiff; application=geotiff; profile=cloud-optimized` | `data`, `elevation` | Digital Surface Model |
| `dtm` | Optional | `image/tiff; application=geotiff; profile=cloud-optimized` | `data`, `elevation` | Digital Terrain Model |
| `multispectral_orthomosaic` | Optional | `image/tiff; application=geotiff; profile=cloud-optimized` | `data` | Multispectral orthomosaic COG |

Example asset definition:

```json
"assets": {
  "metadata": {
    "href": "s3://geonadir-fair-data/datasets/{item_id}/metadata.json",
    "type": "application/json",
    "roles": ["metadata"],
    "title": "Dataset metadata"
  },
  "raw_images": {
    "href": "s3://geonadir-fair-data/datasets/{item_id}/raw-images.zip",
    "type": "application/zip",
    "roles": ["source"],
    "title": "Original raw images"
  },
  "orthomosaic": {
    "href": "s3://geonadir-fair-data/datasets/{item_id}/orthomosaic.tif",
    "type": "image/tiff; application=geotiff; profile=cloud-optimized",
    "roles": ["data"],
    "title": "RGB orthomosaic",
    "eo:bands": [
      {"name": "B1", "common_name": "red"},
      {"name": "B2", "common_name": "green"},
      {"name": "B3", "common_name": "blue"},
      {"name": "B4", "description": "Alpha"}
    ]
  }
}
```

## Collection Summaries

Collection summaries describe expected values or ranges across all items in the
collection. They help users understand what fields exist without inspecting
every item.

Planned summaries for the first public version:

```json
"summaries": {
  "license": ["CC-BY-4.0"],
  "gsd": {
    "minimum": 0.0,
    "maximum": 1.0
  },
  "geonadir:area_m2": {
    "minimum": 0.0,
    "maximum": 100000000.0
  },
  "geonadir:relative_altitude_m": {
    "minimum": 0.0,
    "maximum": 1000.0
  },
  "eo:bands": [
    {
      "name": "B1",
      "common_name": "red"
    },
    {
      "name": "B2",
      "common_name": "green"
    },
    {
      "name": "B3",
      "common_name": "blue"
    },
    {
      "name": "B4",
      "description": "Alpha"
    }
  ],
  "geonadir:iucn_habitat": [
    "Forest",
    "Caves and subterranean",
    "Coastal",
    "Savanna",
    "Desert",
    "Artificial - terrestrial",
    "Shrubland",
    "Neritic",
    "Artificial - aquatic",
    "Grassland",
    "Oceanic",
    "Introduced vegetation",
    "Wetlands",
    "Deep ocean floor",
    "Other",
    "Rocky",
    "Intertidal",
    "Unknown"
  ]
}
```

The numeric range values above are illustrative. The final `minimum` and
`maximum` values will be calculated from the first public publication set. The
band summary describes the common RGB orthomosaic layout; multispectral assets
will define their own band order at asset level.

## STAC Extensions

GeoNadir Fair Data will use STAC extensions only where they add useful,
standardized meaning.

Likely first-pass extensions:

| Extension | Why it is useful |
| --- | --- |
| EO | Describes optical bands such as red, green, blue, red edge, and NIR |
| File | Allows file size and checksum metadata where available |
| Item Assets | Documents the asset keys expected in the collection |

Example:

```json
"stac_extensions": [
  "https://stac-extensions.github.io/eo/v1.1.0/schema.json",
  "https://stac-extensions.github.io/file/v2.1.0/schema.json",
  "https://stac-extensions.github.io/item-assets/v1.0.0/schema.json"
]
```

Projection metadata will not be duplicated in STAC item properties. Each
GeoTIFF contains its own coordinate reference system and geotransform, which
users can inspect directly with GDAL, Rasterio, or QGIS.

## Queryable Fields

The first public STAC API should keep queryables minimal and reliable.

Planned v1 queryables:

| Queryable | Description |
| --- | --- |
| `datetime` | Capture datetime |
| `geonadir:upload_datetime` | Upload or first publication datetime |
| `geonadir:iucn_habitat` | IUCN habitat classification |
| `geometry` | Spatial intersection using bbox-derived geometry |

Optional later queryables:

| Queryable | Description |
| --- | --- |
| `gsd` | Ground sampling distance |
| `geonadir:area_m2` | Captured area in square metres |
| `geonadir:raw_image_count` | Number of source images |
| `geonadir:manufacturer` | Manufacturer, when confidently extracted |

The first version should not expose queryables that GeoNadir cannot populate
consistently.

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

```json
{
  "type": "Feature",
  "stac_version": "1.1.0",
  "id": "{item_id}",
  "collection": "geonadir-fair-data",
  "bbox": [151.0, -34.0, 151.2, -33.8],
  "geometry": {
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
  "properties": {
    "title": "Example GeoNadir UAV survey dataset",
    "description": "Public UAV survey dataset contributed through GeoNadir.",
    "datetime": "2025-04-18T02:14:00Z",
    "created": "2025-04-20T08:31:00Z",
    "updated": "2025-05-02T03:12:00Z",
    "license": "CC-BY-4.0",
    "gsd": 0.026,
    "geonadir:capture_datetime": "2025-04-18T02:14:00Z",
    "geonadir:upload_datetime": "2025-04-20T08:31:00Z",
    "geonadir:modified_datetime": "2025-05-02T03:12:00Z",
    "geonadir:provided_by": "Example Workspace Owner",
    "geonadir:captured_by": "Example field team",
    "geonadir:institution": "Example workspace",
    "geonadir:manufacturer": "DJI",
    "geonadir:camera_model": "FC6310",
    "geonadir:sensor_metadata_source": "image_exif",
    "geonadir:relative_altitude_m": 90.5,
    "geonadir:area_m2": 124300.0,
    "geonadir:raw_image_count": 420,
    "geonadir:iucn_habitat": "Forest",
    "geonadir:geometry_source": "bbox"
  },
  "assets": {
    "metadata": {
      "href": "s3://geonadir-fair-data/datasets/{item_id}/metadata.json",
      "type": "application/json",
      "roles": ["metadata"],
      "title": "Dataset metadata"
    },
    "raw_images": {
      "href": "s3://geonadir-fair-data/datasets/{item_id}/raw-images.zip",
      "type": "application/zip",
      "roles": ["source"],
      "title": "Original raw images"
    },
    "orthomosaic": {
      "href": "s3://geonadir-fair-data/datasets/{item_id}/orthomosaic.tif",
      "type": "image/tiff; application=geotiff; profile=cloud-optimized",
      "roles": ["data"],
      "title": "RGB orthomosaic",
      "eo:bands": [
        {"name": "B1", "common_name": "red"},
        {"name": "B2", "common_name": "green"},
        {"name": "B3", "common_name": "blue"},
        {"name": "B4", "description": "Alpha"}
      ]
    }
  },
  "links": [
    {
      "rel": "collection",
      "href": "https://data.geonadir.com/stac/collections/geonadir-fair-data"
    },
    {
      "rel": "root",
      "href": "https://data.geonadir.com/stac/"
    },
    {
      "rel": "self",
      "href": "https://data.geonadir.com/stac/collections/geonadir-fair-data/items/{item_id}"
    }
  ]
}
```

## Per-dataset `metadata.json` — field → source contract

`metadata.json` **is the STAC Item** for a dataset (minus the server-side `links`,
which the STAC catalog build adds). The direct-S3 notebook reads it as
`props = m.get('properties', m)`, so the same file serves both access paths and
the STAC build is a gather-and-wrap of these files. A committed reference file
lives at [`datasets/geonadir-fair-data/metadata.template.json`](datasets/geonadir-fair-data/metadata.template.json).

Every field is buildable at publish time from one `Dataset` row plus its related
records — no new capture pipeline. Sources below are from `geonadir-backend`.
Legend: **direct** column · **derived** (join/count/parse) · **if-present**
(emit only when populated).

| `metadata.json` field | Source | Kind |
| --- | --- | --- |
| `id` | `Dataset.id` + `Dataset.uav_uid` → `{id}-{uav_uid}` | direct |
| `properties.title` | `Dataset.dataset_name` | direct |
| `properties.description` | `Dataset.description` | direct |
| `properties.datetime` / `geonadir:capture_datetime` | `Dataset.captured_date` | direct |
| `properties.created` / `geonadir:upload_datetime` | `Dataset.created_at` | direct |
| `properties.updated` / `geonadir:modified_datetime` | `Dataset.updated_at` | direct |
| `properties.license` | constant `CC-BY-4.0` | constant |
| `bbox` / `geometry` | `Dataset.extra_json["bbox"]` — **reorder, see traps** | direct + transform |
| `properties.geonadir:geometry_source` | constant `bbox` | constant |
| `properties.gsd` | `Metadata.metadata["gsd"]` | if-present |
| `properties.geonadir:area_m2` | `Metadata.metadata["area_covered"]` — **confirm units** | if-present |
| `properties.geonadir:raw_image_count` | `Dataset.imageupload.count()` (COUNT of `PostImage`) | derived |
| `properties.geonadir:provided_by` | Workspace Owner → `User.full_name` else `username` — **can raise, see traps** | derived |
| `properties.geonadir:captured_by` | `Dataset.data_captured_by` | direct |
| `properties.geonadir:institution` | `Workspace.name` (or `Dataset.institution_name`) | direct |
| `properties.geonadir:iucn_habitat` | `Dataset.category` M2M filtered against the IUCN vocab (`habitat.py`) | derived |
| `properties.geonadir:manufacturer` | parse `EXIF:Make` from `Metadata.metadata` (first image) | if-present |
| `properties.geonadir:camera_model` | parse `EXIF:Model` from `Metadata.metadata` | if-present |
| `properties.geonadir:relative_altitude_m` | parse `XMP:RelativeAltitude` from `Metadata.metadata` | if-present |
| `properties.geonadir:sensor_metadata_source` | constant `image_exif` when the EXIF blob is present | if-present |
| asset presence (`orthomosaic`/`dsm`/`dtm`/`multispectral_orthomosaic`) | `Dataset.extra_json["has_ortho"|"has_dsm"|"has_dtm"|"has_multispec_ortho"]` | direct |
| asset `file:size` | `Dataset.extra_json["ortho_size"|"dsm_size"|"dtm_size"|"multispec_ortho_size"|"raw_images_size"]` — **confirm MB vs bytes** | direct + transform |
| `orthomosaic` `eo:bands` | convention `B1 red, B2 green, B3 blue (, B4 alpha)` from band count | derived |
| `multispectral_orthomosaic` `eo:bands` | `Dataset.band_stats["multispec_ortho"]["band_descr"]` = `{common_name: 1-based index}`; sort by index → `B{i}` + `common_name` | derived |

### Traps to encode in the publisher

1. **bbox is not stored in STAC order.** `extra_json["bbox"]` is the raw
   incoming array; the backend reorders it as `[b[1], b[0], b[3], b[2]]` to build
   the polygon (`core_viewset.py`). STAC `bbox` = that reordered
   `[west, south, east, north]`. Do not pass the stored array through untouched.
   CRS is not recorded — assume WGS84 lon/lat.
2. **Area units.** The key is literally `area_covered`; units are whatever the
   raster-publish lambda posts. Confirm m² before emitting `geonadir:area_m2`.
3. **Size units.** `extra_json` `*_size` values come from the lambda's MB
   calculation (`bytes * 2**-20`). `file:size` (File extension) is **bytes** —
   convert, or drop `file:size` if the source unit can't be verified.
4. **Owner lookup can throw.** `provided_by` uses the `group="Owner"` query that
   raises `MultipleObjectsReturned` on a duplicate active Owner (a known prod
   signature). Catch per-dataset — one bad workspace must not abort the batch.
5. **Habitat vs. user categories.** IUCN habitat is a `Category` M2M row
   indistinguishable from user tags except by string — filter the dataset's
   categories against the known IUCN vocabulary. Result may be 0, 1, or many →
   `geonadir:iucn_habitat` is string-or-array.
6. **Raw-only datasets have no bbox.** `bbox`/`has_*`/`gsd`/`area` are written by
   raster-publish, which runs on the ortho. A public dataset with no processed
   ortho has no bbox → no STAC geometry. Fall back to
   `Dataset.latitude`/`longitude` (promoted from first-image EXIF) as a point, or
   exclude such datasets from the STAC v1 set.
7. **EXIF is first-image only.** `Metadata.metadata` holds the exiftool blob of
   the first uploaded image. Fine for homogeneous surveys; note the assumption.

## Planned Tutorial Notebook

GeoNadir expects to provide a Jupyter notebook with real examples after the
initial application stage.

The notebook will demonstrate how to:

- Open the GeoNadir Fair Data STAC catalog.
- Search by capture date and area of interest.
- Filter by IUCN habitat classification.
- Inspect item metadata and asset links.
- Open an orthomosaic COG directly from S3 when that asset is available.
- Read raster metadata such as CRS, transform, resolution, and band count.
- Display the orthomosaic and compute simple area or pixel summaries when that
  asset is available.
