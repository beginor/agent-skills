# gdal vsi / dataset / mdim / driver

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

`sozip create` options: `--overwrite`, `-r/--recursive`, `-j/--junk-paths`, `--enable-sozip auto|yes|no`, `--sozip-chunk-size`.

## gdal dataset

| Subcommand | Aliases | Purpose |
|------------|---------|---------|
| `copy` | `cp` | Copy all files of a dataset |
| `delete` | `rm`, `remove` | Delete dataset(s) |
| `identify` | — | Report the driver that opens a file |
| `rename` | `ren`, `mv` | Rename dataset files |

```
gdal dataset copy   [OPTIONS] <SOURCE> <DESTINATION>   # --overwrite, -f/--format
gdal dataset delete [OPTIONS] <FILENAME>...            # -f/--format
gdal dataset rename [OPTIONS] <SOURCE> <DESTINATION>   # --overwrite, -f/--format
gdal dataset identify [OPTIONS] <FILENAME>...          # -f json|text, -r, --force-recursive, --report-failures
```

```bash
gdal dataset identify data.tif
gdal dataset identify /data --recursive --of=json
gdal dataset copy data.tif backup.tif --overwrite
```

## gdal mdim

Multidimensional (NetCDF, Zarr, HDF, GRIB) datasets.

| Subcommand | Purpose |
|------------|---------|
| `info` | Information on arrays, dimensions, attributes |
| `convert` | Convert multidimensional dataset |
| `mosaic` | Mosaic multidimensional datasets |

```
gdal mdim info [OPTIONS] <INPUT>
```

`--detailed`, `--array <NAME>`, `--limit <N>`, `--array-option KEY=VALUE`, `--stats`, `--oo`, `--if`.

`gdal mdim --drivers` lists multidimensional drivers as JSON.

```bash
gdal mdim info data.nc --detailed
gdal mdim info data.nc --array temperature --stats
```

## gdal driver

Driver-specific operations.

| Subcommand | Purpose |
|------------|---------|
| `gpkg` | GeoPackage driver operations |
| `gti` | GTI (GDAL Tile Index) driver operations |
| `openfilegdb` | OpenFileGDB driver operations |
| `pdf` | PDF driver operations |

```bash
gdal driver gpkg --help
gdal driver pdf --help
```
