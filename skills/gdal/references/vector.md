# gdal vector

`gdal vector <ACTION> [OPTIONS] <INPUT> <OUTPUT>`

Common options: `-h/--help`, `--json-usage`, `--config KEY=VALUE`, `-q/--quiet`.

Verified against GDAL 3.12.4, 3.13.3 and 3.14.0dev. An option marked **(3.13+)** or **(3.14+)** does not exist in earlier releases; when it replaced an older spelling, that spelling is given in parentheses.

Most writing actions share this option set:

- `-l, --layer, --input-layer` (repeatable) — select input layers
- `-f, --of, --format, --output-format` (GDALG allowed)
- `--co, --creation-option` / `--lco, --layer-creation-option`
- `--overwrite`, `--update`, `--overwrite-layer`
- `--append` xor `--upsert`
- `--output-layer`, `--skip-errors`
- `--no-create-empty-layers` (3.13+)
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
| `combine` | Combine features into collections |
| `compare` | Compare two datasets |
| `concat` | Concatenate datasets |
| `concave-hull` | Concave hull of geometries |
| `convert` | Convert format |
| `convex-hull` | Convex hull of geometries |
| `create` | Create an empty dataset |
| `dissolve` | Dissolve multipart features |
| `edit` | Edit dataset/layer metadata |
| `explode` | Explode fields or geometries |
| `explode-collections` | Explode collection geometries |
| `export-schema` | Export `OGR_SCHEMA` |
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
| `rename-layer` | Rename layers |
| `reproject` | Reproject |
| `segmentize` | Densify segments |
| `select` | Select/exclude fields |
| `set-field-type` | Change a field type |
| `set-geom-type` | Change geometry type |
| `simplify` | Simplify geometries |
| `simplify-coverage` | Simplify shared boundaries |
| `sort` | Spatially order features |
| `sql` | Apply SQL statements |
| `swap-xy` | Swap X and Y |
| `update` | Update a dataset from another one |

`create`, `update`, `combine`, `concave-hull`, `convex-hull`, `dissolve`, `export-schema`, `rename-layer`, and `sort` need GDAL 3.13+; `compare` and `explode` need GDAL 3.14+.

## info

```
gdal vector info [OPTIONS] <INPUT>...
```

- `-f, --of, --format json|text`
- `-l, --layer` (repeatable; conflicts with `--sql`)
- `--features` xor `--summary`
- `--limit <N>` — implies `--features`
- `--sql <stmt>|@file` (conflicts with `--input-layer`)
- `--where <expr>|@file`, `--dialect`
- `--fid <FID>` (3.13+)
- `--crs-format AUTO|WKT2|PROJJSON` (3.13+, text output only)

`--update` exists only in 3.12; from 3.13 on use `gdal vector sql --update` to modify a dataset in place.

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
- `--geometry` / `--geometry-crs` — same, with an arbitrary geometry (3.14+)
- `--bbox-crs` — CRS of the filter box (3.14+)
- `--where <expr>|@file`
- `--update-extent` (3.13+), `--active-layer`

## reproject

```
gdal vector reproject [OPTIONS] <INPUT> <OUTPUT>
```

`-s/--input-crs`, `-d/--output-crs` (3.13+; 3.12 spells them `--src-crs` / `--dst-crs`), `--like <DS>` (optionally with `--like-layer`, both 3.14+), `-l`, `--active-layer`, `--skip-errors`. 3.12/3.13 require `--output-crs`; 3.14 accepts `--output-crs` or `--like`.

## sql

```
gdal vector sql [OPTIONS] <INPUT> [<OUTPUT>]
```

- `--sql <stmt>|@file` (required, repeatable)
- `--output-layer` (repeatable)
- `--dialect OGRSQL|SQLITE|INDIRECT_SQLITE`
- `--update` — modify in place; `--append`, `--upsert`, `--overwrite`
- The input is `-i/--input` in every version; `--dataset` is an extra alias from 3.13 on

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
- `--like <DS>` — template for bounds, CRS and resolution (3.14+)
- `--extent xmin,ymin,xmax,ymax`, `--resolution` xor `--size`, `--tap/--target-aligned-pixels`
- `--optimization AUTO|RASTER|VECTOR`, `--transformer-option`, `--update`

`-b` only works when updating an existing raster; a new output needs `--size` or `--resolution`.

## Geometry operations

| Action | Key options |
|--------|-------------|
| `buffer` | `--distance` (required), `--endcap-style round\|flat\|square`, `--join-style round\|mitre\|bevel`, `--mitre-limit` (5), `--quadrant-segments` (8), `--side both\|left\|right` |
| `simplify` | `--tolerance` (required), `--active-layer`, `--active-geometry` |
| `segmentize` | `--max-length` (required) |
| `make-valid` | `--method linework\|structure`, `--keep-lower-dim` |
| `check-geometry` | `--include-field` (repeatable), `--include-valid`, `--geometry-field` |
| `explode-collections` | `--geometry-type`, `--skip-on-type-mismatch` |
| `set-geom-type` | `--geometry-type`, `--auto` (3.14+), `--layer-only` xor `--feature-only`, `--multi` xor `--single`, `--linear` xor `--curve`, `--dim XY\|XYZ\|XYM\|XYZM`, `--skip` |
| `swap-xy` | `--active-layer`, `--active-geometry` |
| `make-point` | `--x` (required), `--y` (required), `--z`, `--m`, `--output-crs` (3.13+; 3.12 spells it `--dst-crs`) |
| `layer-algebra` | `--operation union\|intersection\|sym-difference\|identity\|update\|clip\|erase` (required), `--method <DS>` (required), `--input-layer`, `--method-layer`, `--geometry-type`, `--input-field`/`--all-input-field`/`--no-input-field`, `--method-field`/`--all-method-field`/`--no-method-field`, `--input-prefix`, `--method-prefix`, `--output-oo` (3.14+) |

## Coverage operations

| Action | Key options |
|--------|-------------|
| `check-coverage` | `--include-valid`, `--geometry-field`, `--maximum-gap-width` |
| `clean-coverage` | `--snapping-distance`, `--merge-strategy longest-border\|max-area\|min-area\|min-index`, `--maximum-gap-width` |
| `simplify-coverage` | `--tolerance` (required), `--preserve-boundary` |

## Attributes and structure

| Action | Key options |
|--------|-------------|
| `select` | `--field` (repeatable), `--geometry`, `--geometry-field` (all three 3.14+; 3.12/3.13 spell the field list `--fields`), `--exclude` xor `--ignore-missing-fields`, `--active-layer`, `--output-layer` |
| `set-field-type` | `--field-name` (required) + `--output-field-type/--field-type` (required); `--input-field-type` (3.13+, 3.12 spells it `--src-field-type`) is declared mutually exclusive with `--field-name`, so in practice only the `--field-name` form works from 3.13 on |
| `edit` | `--geometry-type`, `--crs`, `--metadata`, `--unset-metadata`, `--layer-metadata`, `--unset-layer-metadata`, `--unset-fid`, `--active-layer`, `--output-layer` |
| `concat` | `--mode merge-per-layer-name\|stack\|single`, `--output-layer`, `--source-layer-field-name`, `--source-layer-field-content`, `--field-strategy union\|intersection`, `-s/--input-crs`, `-d/--output-crs` (3.13+; 3.12 spells them `--src-crs` / `--dst-crs`) |
| `partition` | `--field` (repeatable; required in 3.12, optional from 3.13 on), `--scheme hive\|flat`, `--pattern`, `--feature-limit`, `--max-file-size`, `--omit-partitioned-field` |
| `index` | `--recursive`, `--filename-filter`, `--location-name`, `--absolute-path`, `--output-crs` (3.13+; 3.12 spells it `--dst-crs`), `--metadata`, `--nln/--output-layer`, `--source-crs-field-name`, `--source-crs-format`, `--source-layer-name`, `--source-layer-index`, `--accept-different-crs`, `--accept-different-schemas`, `--dataset-name-only` |
| `create` | `-i/--like` xor `--schema` xor `--field NAME:TYPE` (repeatable), `-l`, `--geometry-type`, `--geometry-field`, `--crs`, `--fid`, `--output-layer` |
| `update` | `--input-layer`, `--output-layer`, `--mode merge\|update-only\|append-only`, `--key` (repeatable) |
| `combine` | `--group-by` (repeatable), `--keep-nested`, `--add-extra-fields no\|sometimes-identical\|always-identical` |
| `compare` | `--reference <DS>` (required), `--lax-geometry`, `--skip-all-optional`, `--skip-binary`, `--skip-crs`, `--skip-metadata`, `--skip-fid` |
| `concave-hull` | `--ratio` (required), `--allow-holes`, `--tight` |
| `convex-hull` | `--active-layer`, `--active-geometry` |
| `dissolve` | `--active-layer`, `--active-geometry` |
| `explode` | `--field` (repeatable), `--geometry`, `--geometry-field` (repeatable), `--index-field` |
| `export-schema` | `-l`; `-o` and `--overwrite` need 3.14+ (3.13 always writes to stdout) |
| `rename-layer` | `-l/--input-layer`, `--output-layer`, `--ascii`, `--lower-case`, `--filename-compatible`, `--reserved-characters`, `--replacement-character`, `--max-length` |
| `sort` | `--geometry-field`, `--method hilbert\|strtree`, `--use-tempfile` |

## grid

```
gdal vector grid <SUBCOMMAND> [OPTIONS]
```

Subcommands: `average`, `average-distance`, `average-distance-points`, `count`, `invdist`, `invdistnn`, `linear`, `maximum`, `minimum`, `nearest`, `range`.

```bash
gdal vector grid invdist points.gpkg dem.tif --zfield=z --size=1000,1000
```

Common grid options: `--extent`, `--resolution` xor `--size`, `--ot`, `--crs`, `--bbox`, `-l`, `--sql`, `--zfield`, `--zoffset`, `--zmultiply`, `--power` (2), `--smoothing`, `--radius` xor `--radius1/--radius2`, `--angle`, `--min-points`, `--max-points`, `--min-points-per-quadrant`, `--max-points-per-quadrant`, `--nodata`.
