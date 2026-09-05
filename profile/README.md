# Stratoscale

**Cloud-native geospatial — without the copy.**

We build tools that query Zarr and NetCDF archives with SQL, right where they
sit in object storage. You don't copy anything into a database first.

[stratoscale.io](https://stratoscale.io) · [Blog](https://stratoscale.io/blog) · [hi@stratoscale.io](mailto:hi@stratoscale.io)

---

## Why

Climate and Earth-observation archives already sit in object storage, chunked
and compressed. The usual advice is to copy them into a warehouse before you
can query them. That costs egress, duplicates petabytes, and the copy is stale
the day it lands.

We'd rather send the query to the data. That's what we build.

Everything uses open formats: Zarr, Apache Arrow, Apache DataFusion.

## Projects

### [zarr-datafusion](https://github.com/stratoscale-io/zarr-datafusion) · Rust

A SQL engine for Zarr-native array data, built on
[Apache DataFusion](https://datafusion.apache.org/).

It reads Zarr v2 and v3, and infers the Arrow schema from the store's own
metadata. Filters and projections push down, so a query skips the chunks it
doesn't need. `MIN`, `MAX` and `COUNT` come from statistics without reading any
data. Scans run in parallel and read straight from GCS and S3. VirtualiZarr
reference stores point the same SQL at NetCDF and GRIB files in place.

The CLI has an extended `DESCRIBE` and per-query I/O stats. Larger scans can
spread across several nodes.

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
built on that engine.

`position`, `radius`, `area` and `cube` queries turn into SQL and push down to
chunk reads. Results come back as CoverageJSON, GeoJSON or HTML. With no
configuration it serves the public ARCO-ERA5 store. Give it a TOML file and it
serves as many stores as you list, reading the axes, extents and parameters
from the stores themselves at startup.

Every resource has an HTML page at the same URL as its JSON. A result page's
address is the request that produced it, one `f=` away from CoverageJSON.

## Cookbook

Each recipe is checked against a reference.

- **[ONI from ERA5](https://github.com/stratoscale-io/zarr-datafusion/tree/main/cookbook/el-nino-oni)**
  computes the Oceanic Niño Index for every overlapping three-month season
  since 1950. It's one SQL query over public ERA5 on GCS. In the satellite era
  it tracks NOAA's official table within 0.14 °C on average, at r = 0.96.
- **[NDVI](https://github.com/stratoscale-io/zarr-datafusion/tree/main/cookbook/ndvi)**
  is `(b08 - b04) / (b08 + b04)` over two Sentinel-2 bands, written as a single
  SQL projection. It matches xarray to four decimals.

We write these up on the [blog](https://stratoscale.io/blog).

## Work with us

We're a small senior team specialising in cloud-native array data. We take
hands-on production work. Write to [hi@stratoscale.io](mailto:hi@stratoscale.io).
