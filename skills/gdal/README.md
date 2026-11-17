# GDAL

A Claude Code Skill for the GDAL 3.11+ unified `gdal` command line, covering raster, vector, pipeline, VSI, dataset, and multidimensional workflows.

## What it covers

- **Inspect** datasets with `gdal info` (text or JSON), statistics, histograms, feature listings
- **Convert** between formats: GeoTIFF, COG, GeoPackage, Shapefile, GeoJSON, Parquet, VRT, NetCDF
- **Reproject, clip, mosaic, stack, resize, tile** rasters
- **Reproject, clip, filter, simplify, buffer, rasterize** vectors
- **Raster algebra** with `gdal raster calc`
- **SQL** over vector datasets with `gdal vector sql`
- **Terrain analysis**: hillshade, slope, aspect, TPI, TRI, roughness, viewshed, zonal stats
- **Pipelines** that chain steps with `!` and avoid intermediate files
- **VSI** operations over `/vsicurl/`, `/vsis3/`, `/vsizip/`, and other virtual filesystems

## Installation

```bash
npx skills add beginor/agent-skills --skill gdal
```

## Requirements

GDAL 3.11 or newer with the unified `gdal` command (3.12 recommended). Verify with:

```bash
gdal --version
```

If GDAL is not installed locally, the skill runs commands through the official Docker image:

```bash
docker run --rm -v "$PWD:/data" -w /data ghcr.io/osgeo/gdal:ubuntu-small-latest gdal info dem.tif
```

The image has no entrypoint, so `gdal` must be the first argument after the image name. Mount the directory containing the data and use paths relative to it. The default command is `bash -l`, so add `--entrypoint sh` before the image name to run a shell.

## Layout

```
skills/gdal/
├── SKILL.md                     # Entry point: command map, workflows, conventions
└── references/
    ├── raster.md                # Every `gdal raster` action with options
    ├── vector.md                # Every `gdal vector` action with options
    ├── pipeline.md              # Raster and vector pipeline steps
    └── vsi-dataset.md           # vsi, dataset, mdim, driver groups
```

## Usage examples

```bash
gdal info dem.tif --of=json | jq '.size, .bands | length'
gdal raster convert --format=COG --co COMPRESS=DEFLATE in.tif out.tif
gdal raster reproject --output-crs=EPSG:3857 in.tif out.tif --overwrite
gdal raster mosaic tiles/*.tif mosaic.tif --resolution highest
gdal vector clip --bbox=2,49,3,50 --bbox-crs=EPSG:4326 in.gpkg out.gpkg
gdal pipeline ! read in.tif ! footprint ! buffer 20 ! write out.gpkg --overwrite
```

## Notes

The `gdal` command is still marked provisional upstream, so option names can change between minor releases. Older releases use `--src-*` / `--dst-*`; current releases prefer `--input-*` / `--output-*`. The legacy spellings are still accepted as aliases:

| Legacy | Preferred |
|--------|-----------|
| `--src-crs` | `--input-crs` |
| `--dst-crs` | `--output-crs` |
| `--src-nodata` | `--input-nodata` |
| `--dst-nodata` | `--output-nodata` |
| `--src-min` / `--src-max` | `--input-min` / `--input-max` |
| `--dst-min` / `--dst-max` | `--output-min` / `--output-max` |

Run `gdal <group> <action> --help` on the installed version when a flag is rejected. The Docker image tracks GDAL master, so its version may be newer than the latest stable release.

## License

MIT
