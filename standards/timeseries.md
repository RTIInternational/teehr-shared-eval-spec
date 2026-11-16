# Timeseries columns

- **Status:** agreed 2026-11-15 (TEEHR data model v1).

The names and types a spec can rely on when it reads a timeseries dataset. How each system stores
or partitions them is its own business.

## Dataset-level (fixed for a dataset)

| Field | Values |
| --- | --- |
| `role` | `observed` \| `simulated` \| `forecast` |
| `variable` | the system's variable name |
| `unit` | UCUM |
| `statistic` | `inst` \| `mean` \| `sum` \| `min` \| `max` |
| `period` | a period name (`standards/periods.md`) |

## Columns

| Column | Type | Presence |
| --- | --- | --- |
| `series_key` | string, not null | always |
| `value_time` | timestamp with zone (UTC, µs), not null | always |
| `value` | double | always (null = a missing value) |
| `reference_time` | timestamp with zone, not null | forecasts (issue time) |
| `member` | string, not null | ensembles |
| `quantile` | 4 decimal places, not null | quantile forecasts |
| `quality` | string | optional: the source's own code(s), comma-separated |
| `period_start`, `period_end` | timestamp with zone, not null | `absolute` periods only |

An optional column is either absent from a dataset or never null.

**Key:** `series_key`, [`reference_time`], `value_time`, [`member`], [`quantile`]: the columns
present. Duplicate keys in one upload are refused.

**`quantile`** has 4 decimal places in both systems, so values compare exactly. TEEHR stores
`decimal(5,4)`; VeriRace stores a double rounded to 4 places.

## Other dataset types

- **`layer`**: locations keyed by `location_key`, EPSG:4326 `geometry` (exchanged as in
  `standards/data-api.md`), optional `name`, attributes.
- **`crosswalk`**: `left_key` → `right_key` between two datasets' series, plus optional columns
  (e.g. `weight`).

## Known differences (not shared)

| | TEEHR | VeriRace |
| --- | --- | --- |
| Location layer | per series, with a dataset default | per dataset |
