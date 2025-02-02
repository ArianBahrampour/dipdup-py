# Changelog

Please use [this](https://docs.gitlab.com/ee/development/changelog.html) document as guidelines to keep a changelog.

## 4.0.0-rc1 - [unreleased]

### ⚠ Migration

* Run `dipdup schema approve --hashes` command on every database you want to use with 4.0.0-rc1.

### Added

* cli: Added `dipdup run --skip-hasura` flag to skip updating Hasura metadata.
* cli: Added `dipdip run --early-realtime` flag to establish a realtime connection before all indexes are synchronized.
* cli: Added`dipdup run --merge-subscriptions`  flag to subscribe to all operations/big map diffs during realtime indexing. This flag helps to avoid reaching TzKT subscriptions limit (currently 10000 channels). 
* cli: Added `dipdup status` command to print the current status of indexes from database
* cli: Added `dipdup config export [--unsafe]` command to print config after resolving all links and variables. Add `--unsafe` option to substitute environment variables.
* cli: Added `dipdup cache show` command to get information about file caches used by DipDup.
* cli: Added `dipdup schema approve --hashes` flag to recalculate schema and index config hashes on the next run.
* config: Added `first_level` and `last_level` optional fields to `TemplateIndexConfig`. These limits are applied after ones from the template itself.
* config: Added `daemon` boolean field to `JobConfig` to run a single callback indefinitely. Conflicts with `crontab` and `interval` fields.
* config: Added `advanced` top-level section with following fields:

```yaml
advanced:
  early_realtime: False
  merge_subscriptions: False
  oneshot: False
  postpone_jobs: False
  reindex:
    manual: exception
    migration: wipe
    rollback: ignore
    config_modified: exception
    schema_modified: wipe
  skip_hasura: False
```

`ReindexingRequiredError` exception raised by default when reindexing is triggered. CLI flags have priority over self-titled `AdvancedConfig` fields.

### Fixed

* cli: Fixed crashes and output inconsistency when piping DipDup commands.
* codegen: Fixed missing imports in handlers generated during init.
* coinbase: Fixed possible data inconsistency caused by caching enabled for method `get_candles`.
* http: Fixed increasing sleep time between failed request attempts.
* index: Fixed invocation of head index callback.
* index: Fixed `CallbackError` raised instead of `ReindexingRequiredError` in some cases.
* tzkt: Fixed resubscribing when realtime connectivity is lost for a long time.
* tzkt: Fixed sending useless subscription requests when adding indexes in runtime.
* tzkt: Fixed `get_originated_contracts` and `get_similar_contracts` methods whose output was limited to `HTTPConfig.batch_size` field.
* tzkt: Fixed lots of SignalR bugs by replacing `aiosignalrcore` library with `pysignalr`.

### Deprecated

* cli: `run --oneshot` option is deprecated and will be removed in the next major release. The oneshot mode applies automatically when `last_level` field is set in the index config.
* cli: `clear-cache` command is deprecated and will be removed in the next major release. Use `cache clear` command instead.

### Performance

* config: Configuration files are loaded 10x times faster.
* index: Number of operations processed by matcher reduced by 40%-95% depending on number of addresses and entrypoints used.
* tzkt: Rate limit was increased. Try to set `connection_timeout` to a higher value if requests fail with `ConnectionTimeout` exception.
* tzkt: Improved performance of response deserialization. 

## 3.1.3 - 2021-11-15

### Fixed

* codegen: Fixed missing imports in operation handlers. 
* codegen: Fixed invalid imports and arguments in big_map handlers.

## 3.1.2 - 2021-11-02

### Fixed

* Fixed crash occurred during synchronization of big map indexes.

## 3.1.1 - 2021-10-18

### Fixed

* Fixed loss of realtime subscriptions occurred after TzKT API outage.
* Fixed updating schema hash in `schema approve` command.
* Fixed possible crash occurred while Hasura is not ready.

## 3.1.0 - 2021-10-12

### Added

* New index class `HeadIndex` (configuration: [`dipdup.config.HeadIndexConfig`](https://github.com/dipdup-net/dipdup-py/blob/master/src/dipdup/config.py#L778)). Use this index type to handle head (limited block header content) updates. This index type is realtime-only: historical data won't be indexed during the synchronization stage.
* Added three new commands: `schema approve`, `schema wipe`, and `schema export`. Run `dipdup schema --help` command for details.

### Changed

* Triggering reindexing won't lead to dropping the database automatically anymore. `ReindexingRequiredError` is raised instead. `--forbid-reindexing` option has become default.
* `--reindex` option is removed. Use `dipdup schema wipe` instead.
* Values of `dipdup_schema.reindex` field updated to simplify querying database. See [`dipdup.enums.ReindexingReason`](https://github.com/dipdup-net/dipdup-py/blob/master/src/dipdup/enums.py) class for possible values.

### Fixed

* Fixed `ReindexRequiredError` not being raised when running DipDup after reindexing was triggered.
* Fixed index config hash calculation. Hashes of existing indexes in a database will be updated during the first run.
* Fixed issue in `BigMapIndex` causing the partial loss of big map diffs.
* Fixed printing help for CLI commands.
* Fixed merging storage which contains specific nested structures.

### Improved

* Raise `DatabaseConfigurationError` exception when project models are not compatible with GraphQL.
* Another bunch of performance optimizations. Reduced DB pressure, speeded up parallel processing lots of indexes.
* Added initial set of performance benchmarks (run: `./scripts/run_benchmarks.sh`)

## 3.0.4 - 2021-10-04

### Improved

* A significant increase in indexing speed.

### Fixed

* Fixed unexpected reindexing caused by the bug in processing zero- and single-level rollbacks.
* Removed unnecessary file IO calls that could cause `PermissionError` exception in Docker environments.
* Fixed possible violation of block-level atomicity during realtime indexing.

### Changes

* Public methods of `TzktDatasource` now return immutable sequences.

## 3.0.3 - 2021-10-01

### Fixed

* Fixed processing of single-level rollbacks emitted before rolled back head.

## 3.0.2 - 2021-09-30

### Added

* Human-readable `CHANGELOG.md` 🕺
* Two new options added to `dipdup run` command:
  * `--forbid-reindexing` – raise `ReindexingRequiredError` instead of truncating database when reindexing is triggered for any reason. To continue indexing with existing database run `UPDATE dipdup_schema SET reindex = NULL;`
  * `--postpone-jobs` – job scheduler won't start until all indexes are synchronized. 

### Changed

* Migration to this version requires reindexing.
* `dipdup_index.head_id` foreign key removed. `dipdup_head` table still contains the latest blocks from Websocket received by each datasource.

### Fixed

* Removed unnecessary calls to TzKT API.
* Fixed removal of PostgreSQL extensions (`timescaledb`, `pgcrypto`) by function `truncate_database` triggered on reindex.
* Fixed creation of missing project package on `init`.
* Fixed invalid handler callbacks generated on `init`.
* Fixed detection of existing types in the project.
* Fixed race condition caused by event emitter concurrency.
* Capture unknown exceptions with Sentry before wrapping to `DipDupError`.
* Fixed job scheduler start delay.
* Fixed processing of reorg messages.

## 3.0.1 - 2021-09-24

### Added

* Added `get_quote` and `get_quotes` methods to `TzKTDatasource`.

### Fixed

* Defer spawning index datasources until initial sync is complete. It helps to mitigate some WebSocket-related crashes, but initial sync is a bit slower now.
* Fixed possible race conditions in `TzKTDatasource`.
* Start `jobs` scheduler after all indexes sync with a current head to speed up indexing.
