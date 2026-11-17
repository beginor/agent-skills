# gdal vsi / dataset / crs / mdim / driver

Verified against GDAL 3.12.4, 3.13.3 and 3.14.0dev. An item marked **(3.13+)** or **(3.14+)** does not exist in earlier releases; when it replaced an older spelling, that spelling is given in parentheses.

## gdal vsi

Virtual filesystem operations. Paths use GDAL VSI prefixes: `/vsimem/`, `/vsizip/`, `/vsitar/`, `/vsigzip/`, `/vsicurl/`, `/vsis3/`, `/vsigs/`, `/vsiaz/`, `/vsioss/`, `/vsihdfs/`, `/vsicrypt/`.

| Subcommand | Aliases | Purpose |
|------------|---------|---------|
| `copy` | `cp` | Copy files |
| `delete` | `rm`, `rmdir`, `del` | Delete files/directories |
| `list` | `ls` | List files |
| `move` | `mv`, `ren`, `rename` | Move/rename |
| `sync` | — | Synchronize source and target |
| `sozip` | — | Seek-optimized ZIP (`create`, `list`, `optimize`, `validate`) |

### copy / move

```
gdal vsi copy [OPTIONS] <SOURCE> <DESTINATION>
gdal vsi move [OPTIONS] <SOURCE> <DESTINATION>
```

`copy`: `-r/--recursive`, `--skip-errors`, `-q`.

### list

```
gdal vsi list [OPTIONS] <FILENAME>
```

`-f json|text`, `-l/--long`, `-R/--recursive`, `--depth`, `--abs/--absolute-path`, `--tree`.

### delete

```
gdal vsi delete [OPTIONS] <FILENAME>
```

`-r, -R, --recursive`.

### sync

```
gdal vsi sync [OPTIONS] <SOURCE> <DESTINATION>
```

`-r/--recursive`, `--strategy timestamp|ETag|overwrite` (default `timestamp`), `-j/--num-threads`.

```bash
gdal vsi list /vsizip/data.zip -R --long
gdal vsi copy /vsicurl/https://example.com/a.tif /tmp/a.tif
gdal vsi sync -r /data/ /vsis3/bucket/data/ --strategy ETag
gdal vsi sozip create -i file1.tif -i file2.tif -o archive.zip
```

`sozip create` options: `--overwrite`, `-r/--recursive`, `-j/--junk-paths`, `--no-paths`, `--enable-sozip auto|yes|no`, `--sozip-chunk-size`, `--sozip-min-file-size`, `--content-type`.

## gdal dataset

| Subcommand | Aliases | Purpose |
|------------|---------|---------|
| `check` | — | Report read errors in a dataset (3.13+) |
| `copy` | `cp` | Copy all files of a dataset |
| `delete` | `rm`, `remove` | Delete dataset(s) |
| `identify` | — | Report the driver that opens a file |
| `rename` | `ren`, `mv` | Rename dataset files |

```
gdal dataset check  [OPTIONS] <INPUT>                  # --oo, --if
gdal dataset copy   [OPTIONS] <SOURCE> <DESTINATION>   # --overwrite, -f/--format
gdal dataset delete [OPTIONS] <FILENAME>...            # -f/--format
gdal dataset rename [OPTIONS] <SOURCE> <DESTINATION>   # --overwrite, -f/--format
gdal dataset identify [OPTIONS] <FILENAME>             # -f json|text, -r, --force-recursive, --report-failures
                                                       # 3.13+ adds -o/--output, --co, --lco, -l/--output-layer, --overwrite, --detailed
```

```bash
gdal dataset identify data.tif
gdal dataset identify /data --recursive --of=json
gdal dataset check data.tif
gdal dataset copy data.tif backup.tif --overwrite
```

## gdal crs

CRS inspection (3.14+).

```
gdal crs info [OPTIONS] <INPUT-CRS>
```

`-s/--input-crs` (required), `--validate`, `-i/--identify` (search matching EPSG codes), `--single-line`, `-o/--output-type default|all|wkt_all|wkt|projjson|proj4|wkt1|wkt_esri|wkt2|mapinfo|xml`.

```bash
gdal crs info EPSG:32632 --output-type projjson
gdal crs info "+proj=longlat +datum=WGS84" --identify --validate
```

## gdal mdim

Multidimensional (NetCDF, Zarr, HDF, GRIB) datasets.

| Subcommand | Purpose |
|------------|---------|
| `info` | Information on arrays, dimensions, attributes |
| `convert` | Convert multidimensional dataset |
| `mosaic` | Mosaic multidimensional datasets |
| `reproject` | Reproject a multidimensional dataset (3.14+) |
| `compare` | Compare two multidimensional datasets (3.14+) |
| `pipeline` | Chain `read\|mosaic ! ... ! info\|write` steps (3.14+) |

```
gdal mdim info [OPTIONS] <INPUT>
```

`--detailed`, `--array <NAME>`, `--limit <N>`, `--array-option KEY=VALUE`, `--stats`, `--oo`, `--if`; `--summary` is 3.13+, `-f/--of/--format json|text` is 3.14+.

`gdal mdim convert`: `--array`, `--array-option`, `--group`, `--subset`, `--scale-axes`, `--strict`.
`gdal mdim mosaic`: `--array` (repeatable).
`gdal mdim reproject`: `-d/--output-crs`, `-r/--resampling`.
`gdal mdim compare`: `--reference` (required), `--metric all|none|diff|RMSD|PSNR`, `--array`, `--skip-binary`.

`gdal mdim --drivers` lists multidimensional drivers as JSON.

```bash
gdal mdim info data.nc --detailed
gdal mdim info data.nc --array temperature --stats
gdal mdim pipeline ! read in.nc ! reproject --output-crs=EPSG:32632 ! write out.nc --overwrite
```

## gdal driver

Driver-specific operations. The available subcommands depend on which drivers the build includes; run `gdal driver --help` to see them. The official `ubuntu-small` images ship `gpkg`, `gti`, `openfilegdb` (3.12+), `cog`, `rpftoc` (3.13+) and `zarr`, `icechunk` (3.14+); `parquet` and `pdf` need a build that includes those drivers.

| Driver | Subcommands |
|--------|-------------|
| `cog` | `validate` — check a TIFF is a valid Cloud Optimized GeoTIFF (3.13+) |
| `gpkg` | `repack` — vacuum in place; `validate` — GeoPackage conformance (3.13+) |
| `gti` | `create` — build a GDAL Tile Index |
| `openfilegdb` | `repack` |
| `parquet` | `create-metadata-file` |
| `pdf` | `list-layers` |
| `zarr` | `add-georeferencing-convention` (3.14+) |
| `icechunk` | `list-branches`, `list-tags` (3.14+) |
| `rpftoc` | `create` — build an A.TOC index from CADRG frames (3.13+) |

`gdal driver gti create` gained `--profile`, `--base-url`, `--id-method`, `--id-metadata-item` and `--output-crs` in 3.13; 3.12 spells the CRS option `--dst-crs`.

```bash
gdal driver cog validate out.tif
gdal driver gpkg validate data.gpkg
gdal driver gpkg repack data.gpkg
gdal driver gti create tiles/ index.gti.gpkg
```
