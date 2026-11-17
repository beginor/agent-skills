# gdal pipeline

`gdal pipeline [OPTIONS] <PIPELINE>`

Chains processing steps into one command without intermediate files. Steps are separated by `!`; a leading `!` is optional.

```
read|calc|concat|create|mosaic|stack [READ-OPTIONS] ( ! <STEP-NAME> [STEP-OPTIONS] )* ! write|info|tile|compare [WRITE-OPTIONS]
```

- **Raster first step:** `read`, `calc`, `create`, `mosaic`, `stack`.
- **Raster last step:** `write`, `info`, `tile`, `compare`.
- **Vector first step:** `read`, `concat`.
- **Vector last step:** `write`, `info`, `partition`.
- Steps may repeat except the boundary steps (`read`, `calc`, `mosaic`, `stack`, `compare`, `info`, `tile`, `write`).
- Each step takes the same options as its standalone action, minus input/output.
- Nested pipelines are written in square brackets surrounded by spaces; `tee` writes to a nested pipeline.

Shortcut: `gdal read <FILENAME> ! ...` is `gdal pipeline ! read <FILENAME> ! ...`.

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
| `calc` | `-i` (required), `--calc` (required), `--dialect`, `--flatten`, `--ot`, `--nodata`, `--propagate-nodata`, `--no-check-srs`, `--no-check-extent` |
| `create` | `-i/--like`, `--size`, `--band-count`, `--ot`, `--nodata`, `--burn`, `--crs`, `--bbox`, `--metadata`, `--copy-metadata`, `--copy-overviews` |
| `mosaic` | `-i` (required), `-b`, `--resolution`, `--bbox`, `--target-aligned-pixels`, `--input-nodata`, `--output-nodata`, `--hide-nodata`, `--add-alpha`, `--pixel-function`, `--pixel-function-arg` |
| `stack` | `-i` (required), `-b`, `--resolution`, `--bbox`, `--target-aligned-pixels`, `--input-nodata`, `--output-nodata`, `--hide-nodata` |
| `aspect` | `-b`, `--convention`, `--gradient-alg`, `--zero-for-flat`, `--no-edges` |
| `blend` | `--overlay` (required), `--operator`, `--opacity` |
| `clip` | `--bbox` xor `--window` xor `--geometry` xor `--like`, `--bbox-crs`, `--geometry-crs`, `--like-sql`, `--like-layer`, `--like-where`, `--only-bbox`, `--allow-bbox-outside-source`, `--add-alpha` |
| `color-map` | `-b`, `--color-map`, `--add-alpha`, `--color-selection` |
| `compare` | `--reference` (required), `--skip-*` family |
| `edit` | `-b`, `--crs`, `--bbox`, `--nodata`, `--color-interpretation`, `--color-map`, `--scale`, `--offset`, `--metadata`, `--unset-color-table`, `--unset-metadata`, `--unset-metadata-domain`, `--gcp` |
| `external` | `--command` (required), `-f` |
| `fill-nodata` | `-b`, `-d/--max-distance`, `-s/--smoothing-iterations`, `--mask`, `--strategy` |
| `hillshade` | `-b`, `-z/--zfactor`, `--xscale`, `--yscale`, `--azimuth`, `--altitude`, `--gradient-alg`, `--variant`, `--no-edges` |
| `info` | `-f`, `--mm`, `--stats` xor `--approx-stats`, `--hist`, `--no-gcp`, `--no-md`, `--no-ct`, `--no-fl`, `--checksum`, `--list-mdd`, `--mdd`, `--no-nodata`, `--no-mask`, `--subdataset`, `--crs-format` |
| `materialize` | `-o`, `-f`, `--co`, `--overwrite` |
| `neighbors` | `-b`, `--method`, `--size`, `--kernel`, `--ot`, `--nodata` |
| `nodata-to-alpha` | `--nodata` |
| `overview` | `add` (and its options) |
| `pansharpen` | `--spectral` (required), `-r`, `--weights`, `--nodata`, `--bit-depth`, `--spatial-extent-adjustment`, `-j` |
| `proximity` | `--ot`, `-b`, `--target-values`, `--distance-units`, `--max-distance`, `--fixed-value`, `--nodata` |
| `reclassify` | `-m/--mapping` (required), `--ot` |
| `reproject` | `-s`, `--like`, `-d`, `-r`, `--resolution`, `--size`, `--bbox`, `--bbox-crs`, `-j`, `--target-aligned-pixels`, `--input-nodata`, `--output-nodata`, `--add-alpha`, `--wo`, `--to`, `--et` |
| `resize` | `--resolution` xor `--size`, `-r` |
| `rgb-to-palette` | `--color-count`, `--color-map`, `--output-nodata`, `--no-dither`, `--bit-depth` |
| `roughness` | `-b`, `--no-edges` |
| `scale` | `--ot`, `-b`, `--input-min`, `--input-max`, `--output-min`, `--output-max`, `--exponent`, `--no-clip` |
| `select` | `-b` (required), `--exclude`, `--mask` |
| `set-type` | `--ot` |
| `sieve` | `--mask`, `-b`, `-s/--size-threshold`, `-c` |
| `slope` | `-b`, `--unit`, `--xscale`, `--yscale`, `--gradient-alg`, `--no-edges` |
| `tee` | `--tee-pipeline` |
| `tile` | `-o` (required), `-f`, `--co`, `--tiling-scheme`, `--min-zoom`, `--max-zoom`, `--min-x/--max-x/--min-y/--max-y`, `--no-intersection-ok`, `-r`, `--overview-resampling`, `--convention`, `--tile-size`, `--add-alpha`/`--no-alpha`, `--output-nodata`, `--skip-blank`, `--metadata`, `--copy-src-metadata`, `--aux-xml`, `--kml`, `--resume`, `-j`, `--parallel-method`, `--excluded-values`, `--excluded-values-pct-threshold`, `--nodata-values-pct-threshold`, `--webviewer`, `--url`, `--title`, `--copyright`, `--mapml-template` |
| `tpi` | `-b`, `--no-edges` |
| `tri` | `-b`, `--algorithm`, `--no-edges` |
| `unscale` | `--ot` |
| `update` | `-o` (required), `--geometry`, `--geometry-crs`, `-r`, `--no-update-overviews`, `--wo`, `--to`, `--et` |
| `viewshed` | `-p/--pos`, `-z/--height`, `--sd-filename`, `--target-height`, `--mode`, `--max-distance`, `--min-distance`, `--start-angle`, `--end-angle`, `--high-pitch`, `--low-pitch`, `--curvature-coefficient`, `-b`, `--visible-value`, `--invisible-value`, `--maybe-visible-value`, `--out-of-range-value`, `--output-nodata`, `--observer-spacing`, `-j` |
| `write` | `-o` (required), `-f`, `--co`, `--overwrite`, `--append` |

## Vector steps

| Step | Arguments / options |
|------|---------------------|
| `read` | `-i` (required), `-l`, `--if`, `--oo` |
| `concat` | `-i` (required), `-l`, `--mode`, `--output-layer`, `--source-layer-field-name`, `--source-layer-field-content`, `--field-strategy`, `-s`, `-d` |
| `buffer` | `--distance` (required), `--active-layer`, `--active-geometry`, `--endcap-style`, `--join-style`, `--mitre-limit`, `--quadrant-segments`, `--side` |
| `check-coverage` | `--include-valid`, `--geometry-field`, `--maximum-gap-width` |
| `check-geometry` | `--include-field`, `--include-valid`, `--geometry-field` |
| `clean-coverage` | `--active-layer`, `--snapping-distance`, `--merge-strategy`, `--maximum-gap-width` |
| `clip` | `--active-layer`, `--bbox` xor `--geometry` xor `--like`, `--bbox-crs`, `--geometry-crs`, `--like-sql`, `--like-layer`, `--like-where` |
| `combine` | `--group-by` (repeatable), `--keep-nested`, `--add-extra-fields no\|sometimes-identical\|always-identical` |
| `concave-hull` | `--ratio` (required), `--allow-holes`, `--tight`, `--active-layer`, `--active-geometry` |
| `convex-hull` | `--active-layer`, `--active-geometry` |
| `create` | `-o` (required), `-i/--like` xor `--schema` xor `--field`, `-l`, `-f`, `--co`, `--lco`, `--overwrite`, `--update`, `--overwrite-layer`, `--output-layer`, `--geometry-type`, `--geometry-field`, `--crs`, `--fid` |
| `dissolve` | `--active-layer`, `--active-geometry` |
| `edit` | `--active-layer`, `--output-layer`, `--geometry-type`, `--crs`, `--metadata`, `--unset-metadata`, `--layer-metadata`, `--unset-layer-metadata`, `--unset-fid` |
| `explode-collections` | `--active-layer`, `--active-geometry`, `--geometry-type`, `--skip-on-type-mismatch` |
| `export-schema` | `-l` |
| `external` | `--command` (required), `-f` |
| `filter` | `--active-layer`, `--bbox`, `--where`, `--update-extent` |
| `info` | `-f`, `-l`, `--features` xor `--summary`, `--limit`, `--sql`, `--where`, `--fid`, `--dialect`, `--crs-format` |
| `limit` | `--limit` (required), `--active-layer` |
| `make-point` | `--x` (required), `--y` (required), `--z`, `--m`, `--output-crs` |
| `make-valid` | `--active-layer`, `--active-geometry`, `--method`, `--keep-lower-dim` |
| `materialize` | `-o`, `-f`, `--co`, `--lco`, `--overwrite` |
| `partition` | `-o` (required), `--field`, `--scheme`, `--pattern`, `--feature-limit`, `--max-file-size`, `--omit-partitioned-field`, `--skip-errors` |
| `rename-layer` | `-l`, `--output-layer`, `--ascii`, `--lower-case`, `--filename-compatible`, `--reserved-characters`, `--replacement-character`, `--max-length` |
| `reproject` | `-d/--output-crs` (required), `-s`, `--active-layer` |
| `segmentize` | `--max-length` (required), `--active-layer`, `--active-geometry` |
| `select` | `--fields` (required), `--active-layer`, `--output-layer`, `--exclude` xor `--ignore-missing-fields` |
| `set-field-type` | `--field-name` xor `--input-field-type`, `--output-field-type/--field-type` (required), `--active-layer` |
| `set-geom-type` | `--active-layer`, `--active-geometry`, `--layer-only` xor `--feature-only`, `--geometry-type`, `--multi` xor `--single`, `--linear` xor `--curve`, `--dim`, `--skip` |
| `simplify` | `--tolerance` (required), `--active-layer`, `--active-geometry` |
| `simplify-coverage` | `--tolerance` (required), `--active-layer`, `--preserve-boundary` |
| `sort` | `--geometry-field`, `--method hilbert\|strtree`, `--use-tempfile` |
| `sql` | `--sql` (required, repeatable), `-l/--output-layer`, `--dialect` |
| `swap-xy` | `--active-layer`, `--active-geometry` |
| `tee` | `--tee-pipeline` (one or more) |
| `update` | `-o` (required), `--input-layer`, `--output-layer`, `--mode merge\|update-only\|append-only`, `--key` |
| `write` | `-o` (required), `-f`, `--co`, `--lco`, `--overwrite`, `--update`, `--overwrite-layer`, `--append` xor `--upsert`, `-l/--output-layer`, `--skip-errors`, `--no-create-empty-layers` |
