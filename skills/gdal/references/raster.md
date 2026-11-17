# gdal raster

`gdal raster <ACTION> [OPTIONS] <INPUT> <OUTPUT>`

All actions accept the common options `-h/--help`, `--json-usage`, `--config KEY=VALUE`, and most accept `-q/--quiet`. Every action that writes output accepts `-f/--of/--format`, `--co/--creation-option`, `--overwrite`, `--append`, `--if/--input-format`, and `--oo/--open-option` unless noted.

## Actions

| Action | Purpose |
|--------|---------|
| `as-features` | Create features from pixels |
| `aspect` | Aspect map |
| `blend` | Blend/compose two rasters |
| `calc` | Raster algebra |
| `clean-collar` | Remove collar noise |
| `clip` | Clip to bbox/window/geometry/like |
| `color-map` | Single band → RGB(A) via color map |
| `compare` | Compare two rasters |
| `contour` | DEM → vector contours |
| `convert` | Convert format |
| `create` | Create a new raster |
| `edit` | Edit metadata in place |
| `fill-nodata` | Interpolate nodata regions |
| `footprint` | Raster → footprint polygons |
| `hillshade` | Shaded relief |
| `index` | Vector tile index of rasters |
| `info` | Dataset information |
| `mosaic` | Mosaic (VRT or materialized) |
| `neighbors` | Focal statistics (alias `neighbours`) |
| `nodata-to-alpha` | Nodata → alpha band |
| `overview` | `add` / `delete` / `refresh` overviews |
| `pansharpen` | Pan-sharpening |
| `pipeline` | Multi-step pipeline |
| `pixel-info` | Pixel values at positions |
| `polygonize` | Raster band → polygons |
| `proximity` | Proximity map |
| `reclassify` | Reclassify values |
| `reproject` | Reproject |
| `resize` | Resize pixels, keep extent |
| `rgb-to-palette` | RGB → paletted |
| `roughness` | Roughness map |
| `scale` | Scale band values |
| `select` | Select bands |
| `set-type` | Change data type |
| `shift-longitude` | Shift longitude range (e.g. 0-360 → -180-180) |
| `sieve` | Remove small polygons |
| `slope` | Slope map |
| `stack` | Combine bands into multi-band output |
| `tile` | Generate map tiles |
| `tpi` | Topographic Position Index |
| `tri` | Terrain Ruggedness Index |
| `unscale` | Scaled → unscaled values |
| `update` | Update destination with input content |
| `viewshed` | Viewshed analysis |
| `zonal-stats` | Zonal statistics |

## info

```
gdal raster info [OPTIONS] <INPUT>
```

- `-f, --of, --format json|text` — default JSON (text from 3.12 on the command line)
- `--mm, --min-max` — compute min/max
- `--stats` / `--approx-stats` — full or sampled statistics (mutually exclusive)
- `--hist` — histogram
- `--no-gcp`, `--no-md`, `--no-ct`, `--no-fl` — suppress GCPs, metadata, color table, file list
- `--checksum` — band checksums
- `--list-metadata-domains`, `--mdd <DOMAIN>|all`
- `--no-nodata`, `--no-mask`, `--subdataset <N>`, `--crs-format AUTO|WKT2|PROJJSON`

## convert

```
gdal raster convert [OPTIONS] <INPUT> <OUTPUT>
```

`-f/--format` (GDALG allowed), `--co`, `--overwrite`, `--append`, `--if`, `--oo`.

```bash
gdal raster convert --format=COG --co COMPRESS=JPEG in.tif out.tif
gdal raster convert in.tif out.gpkg
```

## reproject

```
gdal raster reproject [OPTIONS] <INPUT> <OUTPUT>
```

- `-s, --input-crs` / `-d, --output-crs`
- `--like <DATASET>` — template for target bounds, CRS, size and nodata
- `-r, --resampling` (default `nearest`)
- `--resolution xres,yres` xor `--size width,height`
- `--bbox`, `--bbox-crs`, `--target-aligned-pixels`
- `-j, --num-threads` (default `ALL_CPUS`)
- `--input-nodata`, `--output-nodata`, `--add-alpha`
- `--wo/--warp-option`, `--to/--transform-option`, `--et/--error-threshold`

## clip

```
gdal raster clip [OPTIONS] <INPUT> <OUTPUT>
```

One of `--bbox`, `--window col,line,width,height`, `--geometry` (WKT/GeoJSON), `--like <DATASET>`.
Plus `--bbox-crs`, `--geometry-crs`, `--like-sql`, `--like-layer`, `--like-where`, `--only-bbox`, `--allow-bbox-outside-source`, `--add-alpha`.

## calc

```
gdal raster calc [OPTIONS] <INPUTS>... <OUTPUT>
```

- `-i, --input NAME=path` (repeatable; unnamed first input is `X`)
- `--calc <EXPR>` (required, repeatable — one band per expression)
- `--dialect muparser|builtin` (default `muparser`)
- `--flatten` — one band per expression even for multiband input
- `--ot`, `--nodata`, `--propagate-nodata`, `--no-check-crs`, `--no-check-extent`

## mosaic / stack

```
gdal raster mosaic [OPTIONS] <INPUT>... <OUTPUT>
gdal raster stack  [OPTIONS] <INPUT>... <OUTPUT>
```

- `-b, --band` (repeatable), `--absolute-path`
- `--resolution xres,yres|same|average|common|highest|lowest` (default `same`)
- `--bbox`, `--target-aligned-pixels`
- `--input-nodata`, `--output-nodata`, `--hide-nodata`
- mosaic only: `--add-alpha`, `--pixel-function <FN>`, `--pixel-function-arg NAME=VALUE`

`@filename` may be used to pass a list of inputs.

## edit

```
gdal raster edit [OPTIONS] <DATASET>
```

In-place unless `--auxiliary`. Options: `-b/--band`, `--crs`, `--bbox`, `--nodata`, `--color-interpretation`, `--color-map`, `--scale`, `--offset`, `--metadata KEY=VALUE`, `--unset-metadata`, `--unset-metadata-domain`, `--unset-color-table`, `--gcp pixel,line,x,y[,z]` or `@file`, `--stats`, `--approx-stats`, `--hist`.

## overview

```
gdal raster overview add     [OPTIONS] <INPUT>
gdal raster overview delete  [OPTIONS] <DATASET>
gdal raster overview refresh [OPTIONS] <DATASET>
```

`add`: `--levels 2,4,8` xor `--overview-src`, `--external`, `-r/--resampling` (also `gauss`, `average_magphase`), `--min-size`, `--co`.
`delete`: `--external`.
`refresh`: `--external`, `-r`, `--levels`, `--bbox` xor `--like` xor `--use-source-timestamp`.

## tile

```
gdal raster tile [OPTIONS] <INPUT> <OUTPUT-DIR>
```

- `-f/--format` (default `PNG`), `--co`
- `--tiling-scheme raster|WebMercatorQuad|WorldCRS84Quad|...` (default `WebMercatorQuad`)
- `--min-zoom`, `--max-zoom`, `--min-x/--max-x/--min-y/--max-y`
- `-r/--resampling` (default `cubic`), `--overview-resampling`
- `--convention xyz|tms`, `--tile-size`, `--add-alpha`/`--no-alpha`, `--output-nodata`
- `--skip-blank`, `--resume`, `--kml`, `--aux-xml`, `--copy-src-metadata`, `--metadata`
- `--no-intersection-ok` — downgrade "extent outside tile matrix" to a warning
- `-j/--num-threads`, `--parallel-method thread|spawn|fork`
- `--excluded-values`, `--excluded-values-pct-threshold` (50), `--nodata-values-pct-threshold` (100)
- `--webviewer none|all|leaflet|openlayers|mapml|stac` (default `all`), `--url`, `--title`, `--copyright`, `--mapml-template`

## Terrain and analysis

| Action | Key options |
|--------|-------------|
| `hillshade` | `-b`, `-z/--zfactor`, `--xscale`, `--yscale`, `--azimuth` (315), `--altitude` (45), `--gradient-alg Horn\|ZevenbergenThorne`, `--variant regular\|combined\|multidirectional\|Igor`, `--no-edges` |
| `slope` | `-b`, `--unit degree\|percent`, `--xscale`, `--yscale`, `--gradient-alg`, `--no-edges` |
| `aspect` | `-b`, `--convention azimuth\|trigonometric-angle`, `--gradient-alg`, `--zero-for-flat`, `--no-edges` |
| `tpi` / `roughness` | `-b`, `--no-edges` |
| `tri` | `-b`, `--algorithm Riley\|Wilson`, `--no-edges` |
| `viewshed` | `-p/--pos X,Y[,H]`, `-z/--height`, `--sd-filename`, `--target-height`, `--mode normal\|DEM\|ground\|cumulative`, `--max-distance`, `--min-distance`, `--start-angle`, `--end-angle`, `--high-pitch`, `--low-pitch`, `--curvature-coefficient`, `-b`, `--visible-value` (255), `--invisible-value` (0), `--maybe-visible-value` (2), `--out-of-range-value`, `--output-nodata`, `--observer-spacing`, `-j` |
| `zonal-stats` | `--zones <DS>` (required), `--zones-band` xor `--zones-layer`, `--stat <STAT>` (required, repeatable), `--weights`, `--weights-band`, `--pixels default\|fractional\|all-touched`, `--include-field`, `--include-geom`, `--strategy feature\|raster`, `--chunk-size` |
| `proximity` | `--ot` (Float32), `-b`, `--target-values`, `--distance-units pixel\|geo`, `--max-distance`, `--fixed-value`, `--output-nodata` |
| `neighbors` | `-b`, `--method mean\|sum\|min\|max\|stddev\|median\|mode`, `--size`, `--kernel` (required), `--ot`, `--nodata` |
| `fill-nodata` | `-b`, `-d/--max-distance` (100), `-s/--smoothing-iterations`, `--mask`, `--strategy invdist\|nearest` |
| `sieve` | `--mask`, `-b`, `-s/--size-threshold` (2), `-c/--connect-diagonal-pixels` |

`zonal-stats` statistics: `center_x, center_y, count, coverage, frac, max, max_center_x, max_center_y, mean, median, min, minority, min_center_x, min_center_y, mode, stdev, sum, unique, values, variance, variety, weighted_mean, weighted_stdev, weighted_sum, weighted_variance, weights`.

## Conversion between raster and vector

| Action | Key options |
|--------|-------------|
| `polygonize` | `-b` (1), `--attribute-name` (DN), `-c/--connect-diagonal-pixels`, `--output-layer` |
| `contour` | `-b`, `--interval` xor `--levels` xor `-e/--exp-base`, `--off/--offset`, `--elevation-name`, `--min-name`, `--max-name`, `--3d`, `--input-nodata`, `-p/--polygonize`, `--group-transactions`, `--output-layer` |
| `footprint` | `-b`, `--combine-bands union\|intersection`, `--overview` xor `--input-nodata`, `--coordinate-system georeferenced\|pixel`, `--output-crs`, `--split-multipolygons`, `--convex-hull`, `--densify-distance`, `--simplify-tolerance`, `--min-ring-area`, `--max-points` (100), `--location-field` (location), `--no-location-field`, `--absolute-path`, `--output-layer` (footprint) |
| `index` | `--recursive`, `--filename-filter`, `--min-pixel-size`, `--max-pixel-size`, `--location-name`, `--absolute-path`, `--output-crs`, `--metadata`, `--nln/--output-layer`, `--source-crs-field-name`, `--source-crs-format auto\|WKT\|EPSG\|PROJ`, `--profile none\|STAC-GeoParquet`, `--base-url`, `--id-method`, `--id-metadata-item`, `--skip-errors` |
| `as-features` | `-b`, `--geometry-type none\|point\|polygon`, `--skip-nodata`, `--include-xy`, `--include-row-col`, `--output-layer` |
| `pixel-info` | `-p/--pos col,line` or `X,Y` (repeatable), `--position-dataset`, `-l`, `-o/--output`, `-b`, `--overview`, `--position-crs` (default `pixel`), `-r`, `--promote-pixel-value-to-z`, `--include-field` |

## Band manipulation

| Action | Key options |
|--------|-------------|
| `select` | `-b, --band` (required, repeatable; `mask` / `mask:<band>` / color interpretation such as `red`), `--exclude`, `--mask` |
| `set-type` | `--ot` (required) |
| `scale` | `-b`, `--input-min`, `--input-max`, `--output-min`, `--output-max`, `--exponent`, `--no-clip`, `--ot` |
| `unscale` | `--ot` |
| `reclassify` | `-m/--mapping` (required, `@file` allowed), `--keep-color-table`, `--ot` |
| `color-map` | `-b`, `--color-map <FILE>`, `--add-alpha`, `--color-selection interpolate\|exact\|nearest` |
| `rgb-to-palette` | `--color-count` (256), `--color-map`, `--output-nodata`, `--no-dither`, `--bit-depth 5\|8` |
| `nodata-to-alpha` | `--nodata` (repeatable) |
| `resize` | `--size width[,height]` or `%` (required) xor `--resolution`, `-r` |
| `blend` | `--overlay <DS>` (required), `--operator src-over\|hsv-value\|multiply\|screen\|overlay\|hard-light\|darken\|lighten\|color-dodge\|color-burn`, `--opacity` (100) |
| `pansharpen` | `--spectral <DS>...` (required), `-r` (cubic), `--weights`, `--nodata`, `--bit-depth`, `--spatial-extent-adjustment`, `-j` |
| `update` | `--geometry`, `--geometry-crs`, `-r`, `--no-update-overviews`, `--wo`, `--to`, `--et` |
| `compare` | `--reference <DS>` (required), `--metric all\|none\|diff\|RMSD\|PSNR` (default `diff`), `--skip-all-optional`, `--skip-binary`, `--skip-crs`, `--skip-geotransform`, `--skip-overview`, `--skip-metadata`, `--skip-rpc`, `--skip-geolocation`, `--skip-subdataset` |
| `clean-collar` | `--color` (default `black`), `--color-threshold` (15), `--pixel-distance` (2), `--add-alpha`, `--add-mask`, `--algorithm floodfill\|twopasses` |
| `shift-longitude` | `--min-x` (required), `--max-x` (required), `--output-nodata` |

## create

```
gdal raster create [OPTIONS] <OUTPUT>
```

`-f`, `-i/--like`, `--size` xor `--resolution`, `--band-count` (1), `--ot` (Byte), `--nodata`, `--burn` (repeatable), `--crs`, `--bbox`, `--metadata`, `--copy-metadata`, `--copy-overviews`.

```bash
gdal raster create --size 1000,1000 --band-count 3 --ot Byte --burn 0 --crs EPSG:4326 --bbox 2,49,3,50 blank.tif
```
