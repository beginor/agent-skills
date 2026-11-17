# gdal vector

`gdal vector <ACTION> [OPTIONS] <INPUT> <OUTPUT>`

Common options: `-h/--help`, `--json-usage`, `--config KEY=VALUE`, `-q/--quiet`.

Most writing actions share this option set:

- `-l, --layer, --input-layer` (repeatable) — select input layers
- `-f, --of, --format, --output-format` (GDALG allowed)
- `--co, --creation-option` / `--lco, --layer-creation-option`
- `--overwrite`, `--update`, `--overwrite-layer`
- `--append` xor `--upsert`
- `--output-layer`, `--skip-errors`
- `--if/--input-format`, `--oo/--open-option`, `--output-oo/--output-open-option`

Actions that process features also accept `--active-layer` and often `--active-geometry`.

## Actions

| Action | Purpose |
|--------|---------|
| `buffer` | Buffer geometries |
| `check-coverage` | Validate polygon coverage |
| `check-geometry` | Find invalid geometries |
| `clean-coverage` | Remove gaps/overlaps between polygons |
| `clip` | Clip to bbox/geometry/like |
| `concat` | Concatenate datasets |
| `convert` | Convert format |
| `edit` | Edit dataset/layer metadata |
| `explode-collections` | Explode collection geometries |
| `filter` | Filter by bbox / WHERE |
| `grid` | Interpolate scattered points to a grid |
| `index` | Vector tile index |
| `info` | Dataset information |
| `layer-algebra` | union/intersection/clip/erase between layers |
| `make-point` | Build points from X/Y fields |
| `make-valid` | Repair invalid geometries |
| `partition` | Split into multiple files |
| `pipeline` | Multi-step pipeline |
| `rasterize` | Burn geometries into a raster |
| `reproject` | Reproject |
| `segmentize` | Densify segments |
| `select` | Select/exclude fields |
| `set-field-type` | Change a field type |
| `set-geom-type` | Change geometry type |
| `simplify` | Simplify geometries |
| `simplify-coverage` | Simplify shared boundaries |
| `sql` | Apply SQL statements |
| `swap-xy` | Swap X and Y |

There is no `create` action in GDAL 3.12; use `sql --update` or write through `convert`.

## info

```
gdal vector info [OPTIONS] <INPUT>...
```

- `-f, --of, --format json|text`
- `-l, --layer` (repeatable; conflicts with `--sql`)
- `--features` xor `--summary`
- `--limit <N>` — implies `--features`
- `--sql <stmt>|@file` (conflicts with `--input-layer`)
- `--where <expr>|@file`, `--dialect`, `--update`
- `--crs-format AUTO|WKT2|PROJJSON` (text output only)

## convert

```
gdal vector convert [OPTIONS] <INPUT> <OUTPUT>
```

`-l`, `-f`, `--co`, `--lco`, `--overwrite`, `--update`, `--overwrite-layer`, `--append` xor `--upsert`, `--output-layer`, `--skip-errors`.

```bash
gdal vector convert poly.shp out.gpkg
gdal vector convert --update --output-layer=lines line.shp out.gpkg
gdal vector convert --append --output-layer=poly poly2.shp out.gpkg
```

## clip

```
gdal vector clip [OPTIONS] <INPUT> <OUTPUT>
```

One of `--bbox`, `--geometry` (WKT/GeoJSON), `--like <DATASET>`.
Plus `--bbox-crs`, `--geometry-crs`, `--like-layer`, `--like-sql`, `--like-where`, `--active-layer`.

## filter

```
gdal vector filter [OPTIONS] <INPUT> <OUTPUT>
```

- `--bbox xmin,ymin,xmax,ymax` — selects intersecting features, does **not** clip geometry
- `--where <expr>|@file`
- `--update-extent`, `--active-layer`

## reproject

```
gdal vector reproject [OPTIONS] <INPUT> <OUTPUT>
```

`-s/--input-crs`, `-d/--output-crs`, `-l`, `--active-layer`, `--skip-errors`.

## sql

```
gdal vector sql [OPTIONS] <INPUT> [<OUTPUT>]
```

- `--sql <stmt>|@file` (required, repeatable)
- `--output-layer` (repeatable)
- `--dialect OGRSQL|SQLITE|INDIRECT_SQLITE`
- `--update` — modify in place; `--append`, `--upsert`, `--overwrite`

```bash
gdal vector sql in.gpkg out.gpkg --sql "SELECT * FROM country ORDER BY pop DESC" --output-layer sorted
gdal vector sql --update my.gpkg --sql "DELETE FROM countries WHERE pop > 1e6"
gdal vector sql --update my.gpkg --sql "ALTER TABLE countries ADD COLUMN abbrev STRING(10)"
```

## rasterize

```
gdal vector rasterize [OPTIONS] <INPUT> <OUTPUT>
```

- `-b, --band` — bands to burn into
- `--burn <VALUE>` (repeatable) or `-a/--attribute-name <FIELD>` or `--3d`
- `-l/--input-layer` xor `--sql`; `--where`, `--dialect`
- `--invert`, `--all-touched`, `--add`
- `--nodata`, `--init`, `--crs`, `--ot`
- `--extent xmin,ymin,xmax,ymax`, `--resolution` xor `--size`, `--tap/--target-aligned-pixels`
- `--optimization AUTO|RASTER|VECTOR`, `--transformer-option`, `--update`

## Geometry operations

| Action | Key options |
|--------|-------------|
| `buffer` | `--distance` (required), `--endcap-style round\|flat\|square`, `--join-style round\|mitre\|bevel`, `--mitre-limit` (5), `--quadrant-segments` (8), `--side both\|left\|right` |
| `simplify` | `--tolerance` (required), `--active-layer`, `--active-geometry` |
| `segmentize` | `--max-length` (required) |
| `make-valid` | `--method linework\|structure`, `--keep-lower-dim` |
| `check-geometry` | `--include-field` (repeatable), `--include-valid`, `--geometry-field` |
| `explode-collections` | `--geometry-type`, `--skip-on-type-mismatch` |
| `set-geom-type` | `--geometry-type`, `--layer-only` xor `--feature-only`, `--multi` xor `--single`, `--linear` xor `--curve`, `--dim XY\|XYZ\|XYM\|XYZM`, `--skip` |
| `swap-xy` | `--active-layer`, `--active-geometry` |
| `make-point` | `--x` (required), `--y` (required), `--z`, `--m`, `--output-crs` |
| `layer-algebra` | `--operation union\|intersection\|sym-difference\|identity\|update\|clip\|erase` (required), `--method <DS>` (required), `--input-layer`, `--method-layer`, `--geometry-type`, `--input-field`/`--all-input-field`/`--no-input-field`, `--method-field`/`--all-method-field`/`--no-method-field`, `--input-prefix`, `--method-prefix` |

## Coverage operations

| Action | Key options |
|--------|-------------|
| `check-coverage` | `--include-valid`, `--geometry-field`, `--maximum-gap-width` |
| `clean-coverage` | `--snapping-distance`, `--merge-strategy longest-border\|max-area\|min-area\|min-index`, `--maximum-gap-width` |
| `simplify-coverage` | `--tolerance` (required), `--preserve-boundary` |

## Attributes and structure

| Action | Key options |
|--------|-------------|
| `select` | `--fields` (required, repeatable), `--exclude` xor `--ignore-missing-fields` |
| `set-field-type` | `--field-name` xor `--src-field-type`, `--dst-field-type/--field-type` (required) |
| `edit` | `--geometry-type`, `--crs`, `--metadata`, `--unset-metadata`, `--layer-metadata`, `--unset-layer-metadata`, `--unset-fid` |
| `concat` | `--mode merge-per-layer-name\|stack\|single`, `--output-layer`, `--source-layer-field-name`, `--source-layer-field-content`, `--field-strategy union\|intersection`, `-s/--input-crs`, `-d/--output-crs` |
| `partition` | `--field` (required, repeatable), `--scheme hive\|flat`, `--pattern`, `--feature-limit`, `--max-file-size`, `--omit-partitioned-field` |
| `index` | `--recursive`, `--filename-filter`, `--location-name`, `--output-crs`, `--source-crs-field-name`, `--source-crs-format`, `--source-layer-name`, `--source-layer-index`, `--accept-different-crs`, `--accept-different-schemas`, `--dataset-name-only` |

## grid

```
gdal vector grid <SUBCOMMAND> [OPTIONS]
```

Subcommands: `average`, `average-distance`, `average-distance-points`, `count`, `invdist`, `invdistnn`, `linear`, `maximum`, `minimum`, `nearest`, `range`.

```bash
gdal vector grid invdist points.gpkg dem.tif --zfield=z --size=1000,1000
```

Common grid options: `--extent`, `--resolution` xor `--size`, `--ot`, `--crs`, `--bbox`, `--zfield`, `--zoffset`, `--zmultiply`, `--power` (2), `--smoothing`, `--radius` xor `--radius1/--radius2`, `--angle`, `--min-points`, `--max-points`, `--nodata`.
