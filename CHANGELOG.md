# Changelog

## 2026-11-17

- **gdal**
  - 新增 gdal skill，覆盖 GDAL 3.12+ 统一命令行：`info` / `convert` / `raster` / `vector` / `pipeline` / `vsi` / `dataset` / `crs` / `mdim` / `driver`
  - SKILL.md 提供命令地图、通用选项约定、常见工作流；本地无 gdal 时通过 `ghcr.io/osgeo/gdal:ubuntu-small-latest` 镜像执行
  - references 下补充 raster、vector、pipeline、vsi-dataset 四份完整选项参考
  - 选项名统一为 `--input-*` / `--output-*`，并注明旧版 `--src-*` / `--dst-*` 仍作为别名可用
  - 对照 GDAL 3.14 实测补漏：新增 `gdal crs info`、`gdal raster shift-longitude`、`gdal dataset check`，补全 `gdal vector` 的 create/update/combine/compare/dissolve/explode/export-schema/rename-layer/sort 等动作
  - 修正错误选项：`calc --no-check-crs`、`proximity --output-nodata`、`select --field`（`--fields` 已废弃）、`set-field-type --input-field-type`、`vector info` 无 `--update`
  - 修正失效示例：`vector rasterize` 新建栅格需 `--size`/`--resolution` 且不能用 `-b`；`vector select` 改用 `--field`
  - 补全 pipeline 步骤表（as-features、contour、footprint、pixel-info、polygonize、shift-longitude、zonal-stats、grid、layer-algebra、rasterize、info-raster/info-vector、compare-vector 等）
  - 补全 `gdal driver` 子命令（cog/gpkg/gti/openfilegdb/parquet/pdf/zarr/icechunk/rpftoc）与 `gdal mdim` 的 reproject/compare/pipeline
  - 复核版本标注：`vector compare`/`explode` 与 `driver rpftoc create` 的引入版本；修正 pipeline 步骤约束（`calc`/`create` 可重复、`compare` 仅栅格）与 `set-field-type`、`vector reproject` 的互斥关系
  - 对照 3.12.4 / 3.13.3 / 3.14.0dev 三个官方镜像逐项比对 `--help`，为所有跨版本不兼容的动作与选项加上 `(3.13+)` / `(3.14+)` 标注，并注明旧拼写：`--src-*`/`--dst-*` → `--input-*`/`--output-*`（3.13）、`--no-check-srs` → `--no-check-crs`（3.13）、`--fields` → `--field`（3.14）、`proximity --nodata` → `--output-nodata`（3.14）等
  - 修正 `vector export-schema` 的 `-o`/`--overwrite`（3.14+）、`raster create --resolution`（3.14+）、`vector info --update`（仅 3.12）、`driver gpkg validate` 与 `driver gti create` 的新增选项版本
  - 修正 pipeline 步骤表：`clean-collar`/`compare-vector`/`explode`/`layer-algebra`/`shift-longitude` 为 3.14+，`as-features`/`create`/`external`/`pixel-info`/`sort`/`update` 等为 3.13+；`compare` 的 `--skip-*` 展开为完整列表
  - 三镜像实测复核后修正误标：`raster clean-collar` 的选项、`as-features --output-layer`、`raster update --et`、`reproject -s/-d` 在 3.12 已存在，去掉错误的版本标注；`overview add` 的 `-i` 各版本均有，仅 `delete`/`refresh` 的 `-i` 为 3.13+
  - 修正 `vector sql` 输入参数（`--input` 各版本均有，`--dataset` 为 3.13+ 别名）、`vector reproject` 必填项（3.12/3.13 必填 `--output-crs`，3.14 可用 `--like`）、`set-field-type` 实际约束（3.13+ 必须用 `--field-name`，`--input-field-type` 与之互斥）
  - 补充实测发现的约束：`vector partition` 在 3.12 必填 `--field`；pipeline 的 `create` 步骤需自带 `-o`，`update` 步骤要求目标已存在，`external` 步骤需 `--config GDAL_ENABLE_EXTERNAL=YES`
  - 修正示例：`vector clip --like-layer` 不能用于单图层 shapefile；reproject 示例补充 3.12 需用 `--dst-crs` 的说明
  - 兼容性下限由 3.11 改为 3.12（3.11 缺 `gdal dataset`、`raster as-features|blend|compare|zonal-stats` 及多数 `vector` 动作）
  - 文档内全部 66 条示例命令在 3.12.4 / 3.13.3 / 3.14.0dev 三个镜像上逐条实测：3.14 全部通过，3.12/3.13 的失败项与文档标注的版本限制完全一致

## 2026-09-09

- **sharp-db**
  - 在四个元数据提供者的 `GetColumnsQuery()` 原始字符串前添加 `// language=none` 注释，避免 IDE（如 Rider）将内嵌 SQL 误当作 C# 代码解析，消除误报
  - 新增 SQL Server 支持（`--db-type sqlserver`，别名 `mssql`），基于 Microsoft.Data.SqlClient；`tables` / `columns` 支持主键、外键、关联对象、表/列描述及字符串类型长度
  - 新增 SQL Server 复合外键集成测试（`SHARP_DB_SQLSERVER_CONNECTION_STRING` 环境变量，未设置时跳过）
  - 升级 NuGet 包版本：Microsoft.Data.SqlClient 7.0.2、Microsoft.NET.Test.Sdk 18.9.0、Microsoft.Data.Sqlite 10.0.11、MySql.Data 26.7.0、NUnit3TestAdapter 6.3.0
  - 文档同步：README / SKILL.md / CLAUDE.md 更新支持列表与 SQL Server 连接串示例

## 2026-08-19

- **sharp-db**
  - `execute` 新增 `--yes` 非交互开关，Claude Code / 脚本环境下可自动运行 SQL 文件
  - `query` 新增 `--limit <n>` 行数限制（默认 100，`0` 关闭），防止大结果集撑爆上下文，截断时输出提示
  - 所有命令新增 `--connection-env <VAR>`，从环境变量读取连接串，避免明文密码出现在命令行
  - SKILL.md 工作流改为推断优先：从上下文自动推断数据库类型与连接串，仅在无法确定时询问用户
  - 新增多语句 SQLite 文件执行的回归测试，验证 Microsoft.Data.Sqlite 原生批执行行为

## 2026-08-04

- **sharp-db**
  - columns 命令新增 `character_maximum_length` 列，返回字符串类型（char/varchar）的最大长度；PostgreSQL 额外支持 `varchar[]` 数组元素长度提取
  - 文档完善：CLAUDE.md / README.md / SKILL.md 同步 column length 说明，README 修正项目路径

## 2026-07-26

- Add license and readme
- **sharp-db**
  - Move to sharp-db subdirectory
  - add license file

