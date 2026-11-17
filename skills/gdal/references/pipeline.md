# gdal pipeline

`gdal pipeline [OPTIONS] <PIPELINE>`

Chains processing steps into one command without intermediate files. Steps are separated by `!`; a leading `!` is optional.

Verified against GDAL 3.12.4, 3.13.3 and 3.14.0dev. A step marked **(3.13+)** or **(3.14+)** does not exist in earlier releases; an option marked the same way is likewise unavailable there, and where it replaced an older spelling that spelling is given in parentheses.

```
read|calc|concat|create|mosaic|stack [READ-OPTIONS] ( ! <STEP-NAME> [STEP-OPTIONS] )* ! write|info|tile|compare|partition|export-schema [WRITE-OPTIONS]
```

- **Raster first step:** `read`, `calc`, `create`, `mosaic`, `stack`.
- **Raster last step:** `write`, `info`/`info-raster`, `tile`, `compare`.
- **Vector first step:** `read`, `concat`, `create` (3.13+).
- **Vector last step:** `write`, `info`/`info-vector`, `partition`, `compare-vector` (3.14+), `export-schema` (3.13+).
- `info` resolves to its raster or vector variant from the pipeline type; the explicit names are `info-raster` and `info-vector`. `compare` is raster-only — a vector pipeline must spell `compare-vector`.
- `calc` and `create` may be a first or an intermediate step but not the last one; `read`, `concat`, `mosaic`, `stack` are first-only; `write`, `info`, `tile`, `compare`, `partition`, `export-schema` are last-only. All other steps may repeat.
- Each step takes the same options as its standalone action, minus input/output.
- Nested pipelines are written in square brackets surrounded by spaces; `tee` writes to a nested pipeline.

Shortcut: `gdal read <FILENAME> ! ...` is `gdal pipeline read <FILENAME> ! ...`.

## Examples

```bash
gdal pipeline ! read in.tif ! reproject --output-crs=EPSG:3857 ! write out.tif --overwrite
gdal pipeline ! read in.tif ! footprint ! buffer 20 ! write out.gpkg --overwrite
gdal pipeline ! read in.gpkg ! rasterize --size 1000,1000 ! reproject --output-crs=EPSG:4326 ! write out.tif
gdal pipeline ! mosaic tiles/*.tif ! tile out_dir --min-zoom 2 --max-zoom 6
gdal pipeline ! read in.tif ! clip --bbox=2,49,3,50 --bbox-crs=EPSG:4326 ! info --of=json
gdal read in.tif ! info --of=json
```

## Raster steps

| Step | Arguments / options |
|------|---------------------|
| `read` | `-i/--input` (required), `--if`, `--oo` |
| `calc` | `-i` (required), `--calc` (required), `--dialect`, `--flatten`, `--ot`, `--nodata`, `--propagate-nodata`, `--no-check-crs` (3.13+; 3.12 spells it `--no-check-srs`), `--no-check-extent` |
| `create` | `-i/--like`, `--size` xor `--resolution` (3.14+), `--band-count`, `--ot`, `--nodata`, `--burn`, `--crs`, `--bbox`, `--metadata`, `--copy-metadata`, `--copy-overviews` |
| `mosaic` | `-i` (required), `-b`, `--resolution`, `--bbox`, `--target-aligned-pixels`, `--input-nodata` (3.13+), `--output-nodata` (3.13+), `--hide-nodata`, `--add-alpha`, `--pixel-function`, `--pixel-function-arg`; 3.12 spells the nodata pair `--src-nodata` / `--dst-nodata` |
| `stack` | `-i` (required), `-b`, `--resolution`, `--bbox`, `--target-aligned-pixels`, `--input-nodata` (3.13+), `--output-nodata` (3.13+), `--hide-nodata`; 3.12 spells the nodata pair `--src-nodata` / `--dst-nodata` |
| `as-features` | `-b`, `--geometry-type`, `--skip-nodata`, `--include-xy`, `--include-row-col`, `--output-layer` (3.13+) |
| `aspect` | `-b`, `--convention`, `--gradient-alg`, `--zero-for-flat`, `--no-edges` |
| `blend` | `--overlay` (required), `--operator`, `--opacity` |
| `clean-collar` | `--color`, `--color-threshold`, `--pixel-distance`, `--add-alpha`, `--add-mask`, `--algorithm` (3.14+) |
| `clip` | `--bbox` xor `--window` xor `--geometry` xor `--like`, `--bbox-crs`, `--geometry-crs`, `--like-sql`, `--like-layer`, `--like-where`, `--only-bbox`, `--allow-bbox-outside-source`, `--add-alpha` |
| `color-map` | `-b`, `--color-map`, `--add-alpha`, `--color-selection` |
| `compare` | `--reference` (required), `--metric` (3.14+), `--skip-all-optional`, `--skip-binary`, `--skip-crs`, `--skip-geotransform`, `--skip-overview`, `--skip-metadata`, `--skip-rpc`, `--skip-geolocation`, `--skip-subdataset` |
| `contour` | `-b`, `--interval` xor `--levels` xor `-e/--exp-base`, `--off/--offset`, `--elevation-name`, `--min-name`, `--max-name`, `--3d`, `--input-nodata` (3.13+; 3.12 spells it `--src-nodata`), `-p/--polygonize`, `--group-transactions`, `--output-layer` |
| `edit` | `--crs`, `--bbox`, `--nodata`, `--metadata`, `--unset-metadata`, `--unset-metadata-domain`, `--gcp`, `-b` (3.13+), `--color-interpretation` (3.13+), `--color-map` (3.13+), `--scale` (3.13+), `--offset` (3.13+), `--unset-color-table` (3.13+) |
| `external` | `--command` (required), `-f` (3.13+); needs `--config GDAL_ENABLE_EXTERNAL=YES` or the pipeline refuses to run the command |
| `fill-nodata` | `-b`, `-d/--max-distance`, `-s/--smoothing-iterations`, `--mask`, `--strategy` |
| `footprint` | `-b`, `--combine-bands`, `--overview` xor `--input-nodata` (3.13+; 3.12 spells it `--src-nodata`), `--coordinate-system`, `--output-crs` (3.13+; 3.12 spells it `--dst-crs`), `--split-multipolygons`, `--convex-hull`, `--densify-distance`, `--simplify-tolerance`, `--min-ring-area`, `--max-points`, `--location-field`, `--no-location-field`, `--absolute-path`, `--output-layer` |
| `hillshade` | `-b`, `-z/--zfactor`, `--xscale`, `--yscale`, `--azimuth`, `--altitude`, `--gradient-alg`, `--variant`, `--no-edges` |
| `info` | `-f`, `--mm`, `--stats` xor `--approx-stats`, `--hist`, `--no-gcp`, `--no-md`, `--no-ct`, `--no-fl`, `--checksum`, `--list-mdd`, `--mdd`, `--no-nodata`, `--no-mask`, `--subdataset`, `--crs-format` (3.13+) |
| `info-raster` | same options as `info` |
| `info-vector` | `-f`, `-l`, `--features` xor `--summary`, `--limit`, `--sql`, `--where`, `--dialect`, `--fid` (3.13+), `--crs-format` (3.13+) |
| `compare-vector` | `--reference` (required), `--lax-geometry`, `--skip-all-optional`, `--skip-binary`, `--skip-crs`, `--skip-metadata`, `--skip-fid` (3.14+) |
| `export-schema` | `-l`, `-o` (3.14+), `--overwrite` (3.14+) |
| `materialize` | `-o`, `-f`, `--co`, `--overwrite` |
| `neighbors` | `-b`, `--method`, `--size`, `--kernel`, `--ot`, `--nodata` |
| `nodata-to-alpha` | `--nodata` |
| `overview` | `add` (and its options) |
| `pansharpen` | `--spectral` (required), `-r`, `--weights`, `--nodata`, `--bit-depth`, `--spatial-extent-adjustment`, `-j` |
| `pixel-info` | `-b`, `--overview`, `-p/--pos`, `--position-crs`, `-r`; `-i`, `--position-dataset`, `-l`, `--promote-pixel-value-to-z`, `--include-field` are 3.13+ |
| `polygonize` | `-b`, `--attribute-name`, `-c`, `--output-layer` |
| `proximity` | `--ot`, `-b`, `--target-values`, `--distance-units`, `--max-distance`, `--fixed-value`, `--output-nodata` (3.14+; 3.12/3.13 spell it `--nodata`) |
| `reclassify` | `-m/--mapping` (required), `--keep-color-table` (3.14+), `--ot` |
| `reproject` | `-s/--input-crs`, `-d/--output-crs` (3.12 spells the long names `--src-crs`/`--dst-crs`), `--like` (3.13+), `-r`, `--resolution`, `--size`, `--bbox`, `--bbox-crs`, `-j`, `--target-aligned-pixels`, `--input-nodata` (3.13+), `--output-nodata` (3.13+), `--add-alpha`, `--wo`, `--to`, `--et`; 3.12 spells the nodata pair `--src-nodata`/`--dst-nodata` |
| `resize` | `--resolution` xor `--size`, `-r` |
| `rgb-to-palette` | `--color-count`, `--color-map`, `--output-nodata` (3.13+), `--no-dither` (3.13+), `--bit-depth` (3.13+) |
| `roughness` | `-b`, `--no-edges` |
| `scale` | `--ot`, `-b`, `--input-min` (3.13+), `--input-max` (3.13+), `--output-min` (3.13+), `--output-max` (3.13+), `--exponent`, `--no-clip`; 3.12 spells the four as `--src-min`/`--src-max`/`--dst-min`/`--dst-max` |
| `select` | `-b` (required), `--exclude` (3.13+), `--mask` |
| `set-type` | `--ot` |
| `shift-longitude` | `--min-x` (required), `--max-x` (required), `--output-nodata` (3.14+) |
| `sieve` | `--mask`, `-b`, `-s/--size-threshold`, `-c` |
| `slope` | `-b`, `--unit`, `--xscale`, `--yscale`, `--gradient-alg`, `--no-edges` |
| `tee` | `--tee-pipeline` |
| `tile` | `-o` (required), `-f`, `--co`, `--tiling-scheme`, `--min-zoom`, `--max-zoom`, `--min-x/--max-x/--min-y/--max-y`, `--no-intersection-ok`, `-r`, `--overview-resampling`, `--convention`, `--tile-size`, `--add-alpha`/`--no-alpha`, `--output-nodata` (3.13+; 3.12 spells it `--dst-nodata`), `--skip-blank`, `--metadata`, `--copy-src-metadata`, `--aux-xml`, `--kml`, `--resume`, `-j`, `--parallel-method`, `--excluded-values`, `--excluded-values-pct-threshold`, `--nodata-values-pct-threshold`, `--webviewer`, `--url`, `--title`, `--copyright`, `--mapml-template` |
| `tpi` | `-b`, `--no-edges` |
| `tri` | `-b`, `--algorithm`, `--no-edges` |
| `unscale` | `--ot` |
| `update` | `-o` (required), `--geometry`, `--geometry-crs`, `-r`, `--no-update-overviews`, `--wo`, `--to`, `--et` (3.13+) |
| `viewshed` | `-p/--pos`, `-z/--height`, `--sd-filename` (3.13+), `--target-height`, `--mode`, `--max-distance`, `--min-distance`, `--start-angle`, `--end-angle`, `--high-pitch`, `--low-pitch`, `--curvature-coefficient`, `-b`, `--visible-value`, `--invisible-value`, `--maybe-visible-value` (3.13+), `--out-of-range-value`, `--output-nodata` (3.13+; 3.12 spells it `--dst-nodata`), `--observer-spacing`, `-j` |
| `write` | `-o` (required), `-f`, `--co`, `--overwrite`, `--append` |
| `zonal-stats` | `-b`, `--zones` (required), `--zones-band` xor `--zones-layer`, `--weights`, `--weights-band`, `--pixels`, `--stat` (required), `--include-field`, `--strategy`, `--chunk-size`, `-i/--input` (3.13+), `--if` (3.13+), `--oo` (3.13+), `--include-geom` (3.13+) |

## Vector steps

| Step | Arguments / options |
|------|---------------------|
| `read` | `-i` (required), `-l`, `--if`, `--oo` |
| `concat` | `-i` (required), `-l`, `--mode`, `--output-layer`, `--source-layer-field-name`, `--source-layer-field-content`, `--field-strategy`, `-s/--input-crs`, `-d/--output-crs` (3.12 spells the long names `--src-crs` / `--dst-crs`) |
| `buffer` | `--distance` (required), `--active-layer`, `--active-geometry`, `--endcap-style`, `--join-style`, `--mitre-limit`, `--quadrant-segments`, `--side` |
| `check-coverage` | `--include-valid`, `--geometry-field`, `--maximum-gap-width` |
| `check-geometry` | `--include-field`, `--include-valid`, `--geometry-field` |
| `clean-coverage` | `--active-layer`, `--snapping-distance`, `--merge-strategy`, `--maximum-gap-width` |
| `clip` | `--active-layer`, `--bbox` xor `--geometry` xor `--like`, `--bbox-crs`, `--geometry-crs`, `--like-sql`, `--like-layer`, `--like-where` |
| `combine` | `--group-by` (repeatable), `--keep-nested`, `--add-extra-fields no\|sometimes-identical\|always-identical` (3.13+) |
| `concave-hull` | `--ratio` (required), `--allow-holes`, `--tight`, `--active-layer`, `--active-geometry` (3.13+) |
| `convex-hull` | `--active-layer`, `--active-geometry` (3.13+) |
| `create` | `-o` (required, even as a first step), `-i/--like` xor `--schema` xor `--field`, `-l`, `-f`, `--co`, `--lco`, `--overwrite`, `--update`, `--overwrite-layer`, `--output-layer`, `--geometry-type`, `--geometry-field`, `--crs`, `--fid` (3.13+) |
| `dissolve` | `--active-layer`, `--active-geometry` (3.13+) |
| `edit` | `--active-layer`, `--geometry-type`, `--crs`, `--metadata`, `--unset-metadata`, `--layer-metadata`, `--unset-layer-metadata`, `--unset-fid`, `--output-layer` (3.13+) |
| `explode` | `--active-layer`, `--field` (repeatable), `--geometry`, `--geometry-field` (repeatable), `--index-field` (3.14+) |
| `explode-collections` | `--active-layer`, `--active-geometry`, `--geometry-type`, `--skip-on-type-mismatch` |
| `export-schema` | `-l`, `-o` (3.14+), `--overwrite` (3.14+) — the step itself is 3.13+ |
| `external` | `--command` (required), `-f` (3.13+); needs `--config GDAL_ENABLE_EXTERNAL=YES` or the pipeline refuses to run the command |
| `filter` | `--active-layer`, `--bbox`, `--where`, `--update-extent` (3.13+), `--bbox-crs` (3.14+), `--geometry` (3.14+), `--geometry-crs` (3.14+) |
| `grid` | `<COMMAND>` (`average`, `invdist`, `linear`, `nearest`, ...) plus `--extent`, `--resolution` xor `--size`, `--zfield`, `--ot`, `--nodata`; the subcommand is spelled `grid <COMMAND>`, not `grid` alone |
| `info` | `-f`, `-l`, `--features` xor `--summary`, `--limit`, `--sql`, `--where`, `--dialect`, `--fid` (3.13+), `--crs-format` (3.13+) |
| `layer-algebra` | `--operation` (required), `--method` (required), `--input-layer`, `--method-layer`, `--geometry-type`, `--input-field`/`--all-input-field`/`--no-input-field`, `--method-field`/`--all-method-field`/`--no-method-field`, `--input-prefix`, `--method-prefix`, `--output-layer` (3.14+) |
| `limit` | `--limit` (required), `--active-layer` |
| `make-point` | `--x` (required), `--y` (required), `--z`, `--m`, `--output-crs` (3.13+; 3.12 spells it `--dst-crs`) |
| `make-valid` | `--active-layer`, `--active-geometry`, `--method`, `--keep-lower-dim` |
| `materialize` | `-o`, `-f`, `--co`, `--lco`, `--overwrite` |
| `partition` | `-o` (required), `--field`, `--scheme`, `--pattern`, `--feature-limit`, `--max-file-size`, `--omit-partitioned-field`, `--skip-errors`; 3.12 requires `--field`, 3.13+ also accepts `--feature-limit`/`--max-file-size` alone |
| `rasterize` | `-b`, `--burn` xor `-a/--attribute-name` xor `--3d`, `-l` xor `--sql`, `--where`, `--dialect`, `--invert`, `--all-touched`, `--nodata`, `--init`, `--like` (3.14+), `--crs`, `--extent`, `--resolution` xor `--size`, `--tap`, `--ot`, `--optimization`, `--transformer-option` |
| `rename-layer` | `-l`, `--output-layer`, `--ascii`, `--lower-case`, `--filename-compatible`, `--reserved-characters`, `--replacement-character`, `--max-length` (3.13+) |
| `reproject` | `-d/--output-crs`, `-s/--input-crs` (3.12 spells the long names `--src-crs` / `--dst-crs`), `--active-layer`; `--output-crs` is required in 3.12/3.13, while 3.14 accepts `--output-crs` or `--like` (3.14+, optionally with `--like-layer`, 3.14+) |
| `segmentize` | `--max-length` (required), `--active-layer`, `--active-geometry` |
| `select` | `--field` (repeatable, 3.14+), `--geometry` (3.14+), `--geometry-field` (3.14+), `--active-layer`, `--output-layer`, `--exclude` xor `--ignore-missing-fields`; 3.12/3.13 spell the field list `--fields`, still accepted in 3.14 but deprecated and removed in 3.15 |
| `set-field-type` | `--field-name` (required) + `--output-field-type/--field-type` (required), `--active-layer`; `--input-field-type` (3.13+, 3.12 spells it `--src-field-type`) is mutually exclusive with `--field-name`, so only the `--field-name` form works from 3.13 on; 3.12 spells the target type `--dst-field-type` |
| `set-geom-type` | `--active-layer`, `--active-geometry`, `--layer-only` xor `--feature-only`, `--geometry-type`, `--auto` (3.14+), `--multi` xor `--single`, `--linear` xor `--curve`, `--dim`, `--skip` |
| `simplify` | `--tolerance` (required), `--active-layer`, `--active-geometry` |
| `simplify-coverage` | `--tolerance` (required), `--active-layer`, `--preserve-boundary` |
| `sort` | `--geometry-field`, `--method hilbert\|strtree`, `--use-tempfile` (3.13+) |
| `sql` | `--sql` (required, repeatable), `-l/--output-layer`, `--dialect` |
| `swap-xy` | `--active-layer`, `--active-geometry` |
| `tee` | `--tee-pipeline` (one or more) |
| `update` | `-o` (required, must be an existing dataset — the step updates it in place), `--input-layer`, `--output-layer`, `--mode merge\|update-only\|append-only`, `--key` (3.13+) |
| `write` | `-o` (required), `-f`, `--co`, `--lco`, `--overwrite`, `--update`, `--overwrite-layer`, `--append` xor `--upsert`, `-l/--output-layer`, `--skip-errors`, `--no-create-empty-layers` (3.13+) |
