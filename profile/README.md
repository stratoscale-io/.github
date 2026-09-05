# Stratoscale

**Cloud-native geospatial — without the copy.**

Query petabyte-scale Zarr and NetCDF archives with SQL, directly in object
storage. No pipelines to copy the data into a database first.

[stratoscale.io](https://stratoscale.io) · [Blog](https://stratoscale.io/blog) · [hi@stratoscale.io](mailto:hi@stratoscale.io)

---

## Why

Earth-observation and climate archives are already chunked, compressed, and
sitting in object storage. The usual next step — copy them into a warehouse so
they can be queried — costs egress, duplicated petabytes, and a copy that goes
stale the moment it lands. We build the engine and the servers that let you
skip that step.

Open formats, permissively licensed: Zarr, Apache Arrow, Apache DataFusion.

## Projects

### [zarr-datafusion](https://github.com/stratoscale-io/zarr-datafusion) · Rust

SQL on Zarr-native array data, powered by
[Apache DataFusion](https://datafusion.apache.org/). Zarr v2 and v3, Arrow
schema inferred from store metadata, projection/limit/filter pushdown that
prunes chunks before they are read, `MIN`/`MAX`/`COUNT` answered from
statistics, chunk-level parallelism, and reads straight from GCS and S3.
VirtualiZarr reference stores put the same SQL over NetCDF and GRIB byte
ranges. Ships an interactive CLI with an extended `DESCRIBE` and per-query I/O
statistics; optional multi-node execution via `datafusion-distributed`.

```sql
CREATE EXTERNAL TABLE era5 STORED AS ZARR
LOCATION 'gs://gcp-public-data-arco-era5/ar/model-level-1h-0p25deg.zarr-v1';

SELECT latitude, longitude, AVG(2m_temperature)
FROM era5
WHERE time > '2020-01-01'
GROUP BY latitude, longitude;
```

### [ogc-edr](https://github.com/stratoscale-io/ogc-edr) · Rust

An [OGC API - Environmental Data Retrieval](https://ogcapi.ogc.org/edr/) server
over that engine. `position`, `radius`, `area` and `cube` queries become SQL
pushed down to chunk reads, returned as CoverageJSON, GeoJSON or HTML. Serves
the public ARCO-ERA5 store with no configuration, or any number of Zarr stores
from a TOML file — axes, extents, resolution and every parameter are read from
the stores themselves at startup.

Every resource has an HTML representation on the same URL as its JSON, so a
result page's address *is* the API request that produced it, one `f=` away from
CoverageJSON.

## Cookbook

Recipes validated against the reference, not toy examples:

- **[ONI from ERA5](https://github.com/stratoscale-io/zarr-datafusion/tree/main/cookbook/el-nino-oni)** —
  the Oceanic Niño Index for every overlapping 3-month season since 1950, as one
  SQL query over public ERA5 on GCS, checked against NOAA's official table:
  MAE 0.14 °C in the satellite era, Pearson r 0.96, weighted κ 0.85.
- **[NDVI](https://github.com/stratoscale-io/zarr-datafusion/tree/main/cookbook/ndvi)** —
  `(b08 - b04) / (b08 + b04)` across two co-registered Sentinel-2 bands as a
  single SQL projection, matching xarray to four decimals.

Write-ups on the [blog](https://stratoscale.io/blog).

## Work with us

Small senior team, hands-on production engagements, deep specialisation in
cloud-native array data. → [hi@stratoscale.io](mailto:hi@stratoscale.io)
