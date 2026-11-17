---
name: gdal
description: Process geospatial data with the GDAL 3.12+ unified CLI (`gdal`). Use when the user wants to inspect, convert, reproject, clip, mosaic, tile, rasterize, or run SQL on raster/vector datasets (GeoTIFF, COG, GeoPackage, Shapefile, Parquet, VRT, NetCDF, ...), or chain such steps in a pipeline. Covers `gdal info`, `gdal raster *`, `gdal vector *`, `gdal pipeline`, `gdal vsi`, `gdal dataset`, `gdal crs`, `gdal mdim`, and `gdal driver`.
compatibility: requires GDAL 3.12+ with the unified `gdal` command (3.14 recommended; 3.12 lacks `gdal crs`, `gdal dataset check`, `gdal vector create`, `gdal raster shift-longitude` and spells many options `--src-*`/`--dst-*`)
license: MIT
---

# GDAL

GDAL's unified CLI replaces the legacy `gdal_translate` / `gdalwarp` / `ogr2ogr` / `gdalinfo` / `gdaladdo` tools with one `gdal` command built from subcommands. Everything is `gdal <group> <action> [OPTIONS] <INPUT> <OUTPUT>`.

## Running GDAL

Check for a local binary first:

```bash
gdal --version
```

If `gdal` is not installed locally, run it through the official Docker image. The image has no entrypoint, so `gdal` must be the first argument after the image name:

```bash
docker run --rm ghcr.io/osgeo/gdal:ubuntu-small-latest gdal --version
```

Mount the directory holding the data and set it as the working directory, then use paths relative to that mount:

```bash
docker run --rm -v "$PWD:/data" -w /data ghcr.io/osgeo/gdal:ubuntu-small-latest gdal info dem.tif
docker run --rm -v "$PWD:/data" -w /data ghcr.io/osgeo/gdal:ubuntu-small-latest gdal raster convert --format=COG in.tif out.tif
```

For data outside the current directory, mount that directory instead (`-v /path/to/data:/data`). For multi-step work, prefer one container with `&&`-chained commands over several starts:

```bash
docker run --rm -v "$PWD:/data" -w /data --entrypoint sh ghcr.io/osgeo/gdal:ubuntu-small-latest \
  -c 'gdal raster reproject --output-crs=EPSG:3857 in.tif tmp.tif && gdal raster convert --format=COG tmp.tif out.tif'
```

The image's default command is `bash -l`, so `--entrypoint sh` (or `--entrypoint bash`) is needed to run a shell. The image tracks GDAL master, so its version may be newer than the latest stable release.

## Command map

| Group | Purpose |
|-------|---------|
| `gdal info <INPUT>` | Shortcut for `gdal raster info` or `gdal vector info` |
| `gdal convert <IN> <OUT>` | Shortcut for `gdal raster convert` or `gdal vector convert` |
| `gdal raster <action>` | Raster processing (convert, reproject, clip, mosaic, calc, tile, ...) |
| `gdal vector <action>` | Vector processing (convert, clip, filter, reproject, sql, rasterize, ...) |
| `gdal pipeline <steps>` | Chain steps with `!` in one command |
| `gdal vsi <action>` | Virtual filesystem ops (copy, list, delete, move, sync, sozip) |
| `gdal dataset <action>` | Dataset-level ops (check, copy, delete, identify, rename) |
| `gdal mdim <action>` | Multidimensional datasets (info, convert, mosaic; reproject/compare/pipeline are 3.14+) |
| `gdal crs info <CRS>` | Inspect, validate, or identify a CRS (3.14+) |
| `gdal driver <name> <action>` | Driver-specific ops (cog, gpkg, gti, openfilegdb, parquet, pdf, zarr, ...) |

Shortcuts: `gdal <FILENAME>` is `gdal info <FILENAME>`; `gdal read <FILENAME> ! ...` is `gdal pipeline read <FILENAME> ! ...`.

Useful discovery commands:

```bash
gdal --version
gdal --drivers                                  # all drivers as JSON
gdal raster --help                              # list raster actions
gdal raster convert --help                      # options for one action
gdal raster convert --json-usage                # machine-readable usage
```

## Conventions shared by almost every action

- `-i, --input` / `-o, --output` — input and output datasets (often also positional).
- `-f, --of, --format, --output-format` — output format, e.g. `GTiff`, `COG`, `GPKG`, `GeoJSON`, `Parquet`. Inferred from the extension when omitted.
- `--co, --creation-option KEY=VALUE` — driver creation options, repeatable (`--co COMPRESS=DEFLATE`).
- `--lco, --layer-creation-option KEY=VALUE` — vector layer creation options.
- `--overwrite` — replace existing output (required to overwrite; mutually exclusive with `--append`).
- `--append` — add the input as a subdataset of an existing output.
- `-q, --quiet` — suppress progress bar.
- `--config KEY=VALUE` — GDAL configuration option, repeatable.
- `--if, --input-format` / `--oo, --open-option` — restrict candidate drivers / pass driver open options.
- `--bbox xmin,ymin,xmax,ymax` — extent in the target CRS; `--bbox-crs` sets the CRS of those numbers.
- `--like <DATASET>` — use another dataset as a template for bounds/CRS/size.
- `-r, --resampling` — `nearest|bilinear|cubic|cubicspline|lanczos|average|rms|mode|min|max|med|q1|q3|sum`.
- `--ot, --datatype` — output data type (`Byte|Int16|UInt16|Int32|Float32|Float64|...`).
- `--input-nodata` / `--output-nodata` — input/output nodata values (3.13+; 3.12 spells them `--src-nodata` / `--dst-nodata`).
- `-s, --input-crs` / `-d, --output-crs` — CRS for reprojection (3.13+; 3.12 spells them `--src-crs` / `--dst-crs`).

Version markers in this skill: **(3.13+)** and **(3.14+)** mean the option or action is absent from earlier releases. Verified against GDAL 3.12.4, 3.13.3 and 3.14.0dev.

## Common workflows

### Inspect

```bash
gdal info data.tif                              # text summary
gdal info data.tif --of=json | jq .             # JSON
gdal raster info data.tif --stats --hist        # force statistics + histogram
gdal vector info data.gpkg --summary            # layer names + geometry types
gdal vector info data.gpkg --features --limit 5 # first 5 features
gdal vector info data.gpkg --sql "SELECT count(*) FROM layer"
gdal crs info EPSG:32632 --identify --output-type projjson
```

### Convert / optimize

```bash
gdal raster convert --format=COG --co COMPRESS=DEFLATE in.tif out.tif
gdal raster convert in.tif out.gpkg             # raster into GeoPackage
gdal vector convert poly.shp out.gpkg           # Shapefile -> GeoPackage
gdal vector convert --lco SPATIAL_INDEX=YES in.gpkg out.gpkg
```

### Reproject

```bash
gdal raster reproject --output-crs=EPSG:3857 in.tif out.tif --overwrite
gdal raster reproject --output-crs=EPSG:32632 -r bilinear --resolution 10,10 in.tif out.tif
gdal vector reproject --output-crs=EPSG:4326 in.gpkg out.gpkg --overwrite
```

`--output-crs` is the 3.13+ spelling; on 3.12 it is rejected, use `--dst-crs` (which also works as a hidden alias in 3.13/3.14). Same for `--input-crs`/`--src-crs`.

### Clip

```bash
gdal raster clip --bbox=2,49,3,50 --bbox-crs=EPSG:4326 in.tif out.tif --overwrite
gdal raster clip --like=reference.tif in.tif out.tif --overwrite
gdal raster clip --geometry="POLYGON((2 49,3 49,3 50,2 50,2 49))" in.tif out.tif
gdal vector clip --bbox=2,49,3,50 --bbox-crs=EPSG:4326 in.gpkg out.gpkg --overwrite
gdal vector clip --like countries.gpkg --like-layer France in.gpkg out.gpkg
```

`gdal vector filter --bbox` selects features intersecting the box **without** clipping their geometry; use `clip` when geometry must be cut.

### Mosaic / stack

```bash
gdal raster mosaic tiles/*.tif mosaic.tif --resolution highest --overwrite
gdal raster mosaic --format COG --co BIGTIFF=YES *.tif mosaic.tif
gdal raster stack red.tif green.tif blue.tif rgb.tif
gdal raster stack raster1.tif raster2.tif result.vrt
```

`mosaic` places inputs side by side; `stack` combines bands into a multi-band output. Writing `.vrt` produces a virtual dataset.

### Raster algebra

```bash
gdal raster calc -i A=a.tif -i B=b.tif -o out.tif --calc "A+B"
gdal raster calc -i input.tif -o out.tif --calc "X+3"
gdal raster calc -i A=a.tif -o out.tif --calc="A > 0 ? A : NaN"
gdal raster calc -i A=a.tif -i B=b.tif -o out.tif --flatten --calc=mean --dialect=builtin
```

Inputs are named with `NAME=path`; the first unnamed input is `X`. `--calc` is repeatable (one output band per expression).

### Vector SQL and attributes

```bash
gdal vector sql in.gpkg out.gpkg --sql "SELECT * FROM layer WHERE pop > 1000"
gdal vector sql --update data.gpkg --sql "ALTER TABLE layer ADD COLUMN code STRING(10)"
gdal vector filter --where='pop > 1000' in.gpkg out.gpkg --overwrite
gdal vector select in.gpkg out.gpkg --field name,pop   # 3.14+; 3.12/3.13 use --fields
gdal vector select in.gpkg out.gpkg --field pop --exclude
```

### Rasterize / vectorize

```bash
gdal vector rasterize --burn 255 --size 1000,1000 -l mask mask.shp mask.tif
gdal vector rasterize -a ROOF_H --where "class='A'" -l footprints footprints.shp dem.tif --update
gdal raster polygonize in.tif out.gpkg
gdal raster contour dem.tif contours.gpkg --interval 10
```

### Terrain and analysis

```bash
gdal raster hillshade dem.tif hillshade.tif --azimuth 315 --altitude 45
gdal raster slope dem.tif slope.tif --unit percent
gdal raster aspect dem.tif aspect.tif
gdal raster tpi dem.tif tpi.tif
gdal raster tri dem.tif tri.tif
gdal raster roughness dem.tif rough.tif
gdal raster viewshed dem.tif viewshed.tif --pos 500,500,1.8
gdal raster zonal-stats dem.tif stats.gpkg --zones zones.gpkg --stat mean,min,max
```

### Overviews and tiles

```bash
gdal raster overview add in.tif --levels 2,4,8,16 -r average
gdal raster overview delete in.tif
gdal raster shift-longitude global.tif shifted.tif --min-x -180 --max-x 180   # 3.14+
gdal raster tile in.tif tiles/ --min-zoom 2 --max-zoom 5 --skip-blank
gdal raster tile --tiling-scheme raster --format COG --tile-size 4096 in.tif tiles/
```

### Pipeline

```bash
gdal pipeline ! read in.tif ! reproject --output-crs=EPSG:3857 ! write out.tif --overwrite
gdal pipeline ! read in.tif ! footprint ! buffer 20 ! write out.gpkg --overwrite
gdal pipeline ! read in.gpkg ! rasterize --size 1000,1000 ! reproject --output-crs=EPSG:4326 ! write out.tif
gdal read in.tif ! info --of=json
```

First step: `read|calc|concat|create|mosaic|stack`. Last step: `write|info|tile|compare|partition|export-schema`. Steps are separated by `!`; the leading `!` is optional. Steps use the same options as their standalone action, minus input/output. `create` as a first step and `export-schema` as a last step need 3.13+; `compare-vector` needs 3.14+; everything else works in 3.12. Step options follow the same version rules as the standalone actions, so `reproject` needs `--dst-crs` on 3.12. The vector `create` step still needs its own `-o`, and the `external` step only runs with `--config GDAL_ENABLE_EXTERNAL=YES`.

### VSI and dataset management

```bash
gdal vsi list /vsizip/data.zip -R --long
gdal vsi copy /vsicurl/https://example.com/a.tif /tmp/a.tif
gdal vsi sync -r /data/ /vsicurl/https://example.com/data/ --strategy ETag
gdal dataset identify data.tif
gdal dataset copy data.tif backup.tif
gdal dataset delete data.tif
```

## Reference files

Load these only when you need the full option list for a specific action:

- `references/raster.md` — every `gdal raster` action with options
- `references/vector.md` — every `gdal vector` action with options
- `references/pipeline.md` — pipeline steps for raster and vector
- `references/vsi-dataset.md` — `vsi`, `dataset`, `crs`, `mdim`, `driver` groups

## Workflow

1. **Locate the binary** — run `gdal --version` locally; if missing, prefix commands with `docker run --rm -v "$PWD:/data" -w /data ghcr.io/osgeo/gdal:ubuntu-small-latest`.
2. **Inspect before transforming** — run `gdal info` (or `--of=json` piped to `jq`) to learn driver, size, CRS, bands, nodata, and layer names. Never guess a CRS or band count.
3. **Pick the narrowest action** — `convert` for format changes, `reproject` for CRS, `clip` for extent, `mosaic`/`stack` for combining, `calc` for arithmetic, `sql`/`filter` for attributes.
4. **Always pass `--overwrite`** when the output may exist; without it GDAL refuses to write.
5. **Prefer one pipeline** over chained intermediate files when several steps are needed.
6. **Verify the result** — run `gdal info` on the output and report size, CRS, band count, and nodata.

## Notes

- `--format` values are driver short names (`GTiff`, `COG`, `GPKG`, `GeoJSON`, `Parquet`, `VRT`, `PNG`, `JPEG`, `NetCDF`, `Zarr`). Run `gdal --drivers` for the full list.
- `GDALG` is accepted as an output format to write a streamed algorithm definition instead of materializing pixels.
- `--append` adds the input as a subdataset for rasters (GeoTIFF, GeoPackage) and appends features to an existing layer for vectors; it creates the target if absent.
- `gdal vector create` (3.13+) builds an empty dataset from `--like`, `--schema`, or `--field NAME:TYPE`; older releases need `gdal vector sql --update` or `convert`.
- `gdal raster overview add` modifies the dataset in place unless `--external` is given.
- `gdal raster edit` and `gdal raster overview add/delete/refresh` update files in place — confirm with the user before running them on data they did not ask to modify.
- Exit code is 0 on success; warnings are still success. Non-zero means a real error.
- Actions added after 3.12: `gdal vector create|update|combine|concave-hull|convex-hull|dissolve|export-schema|rename-layer|sort`, `gdal dataset check`, `gdal dataset identify -o/--detailed`, `gdal driver cog validate`, `gdal driver gpkg validate`, `gdal driver rpftoc create`, `gdal mdim info --summary` (3.13); `gdal vector compare|explode`, `gdal raster shift-longitude`, `gdal crs info`, `gdal mdim reproject|compare|pipeline`, `gdal driver zarr`, `gdal driver icechunk` (3.14). Check `--help` before relying on them.
- `gdal` is still marked provisional by GDAL, so option names can shift between minor releases — check `--help` on the installed version when a flag is rejected. The `--src-*` / `--dst-*` spellings (`--src-crs`, `--src-nodata`, `--src-min`, `--no-check-srs`, `--src-field-type`, proximity `--nodata`) were renamed to `--input-*` / `--output-*` in 3.13 but are still accepted as hidden aliases in 3.12, 3.13 and 3.14; prefer the new names.
- The Docker image tracks GDAL master, so its version can be newer than the latest stable release. Run `gdal --version` inside the container before relying on a recently added action.
