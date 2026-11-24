# Evaluation spec (v0)

- **Status:** draft v0. Seeded from TEEHR's `spec-schema-v0.md`, which follows VeriRace's evaluation
  spec v1 with TEEHR naming.
- A JSON Schema (`spec/schema.json`) comes next, starting from VeriRace's
  `evaluation-spec-v1.schema.json`.

A spec is a **serializable, versioned** description of an evaluation. It is parsed and validated
into a typed object, never string-interpolated into SQL. The same spec runs on either system and
gives the same result.

## Design rules

- **Declarative.** The spec says *what* to compute; the engine decides *how*.
- **A source is a dataset.** Each dataset is homogeneous in variable, statistic and period, so an
  input names the dataset directly. A `filter` narrows within it by time, location, member,
  approval status or qualifier, never by variable.
- **Validated before any run** against the system's datasets, columns, metrics and units.
- **Units are explicit** on every unit-bearing parameter (`standards/units.md`).
- **Versioned.** `version` is recorded on every run.

## Steps (always in this order)

`inputs → pair → fields → filter → aggregate → output`

## Example

```yaml
version: 1
name: nwm30-vs-usgs-by-lead-time

inputs:
  primary: {dataset: usgs_streamflow}
  secondary:
    - {dataset: nwm30_short_range}
    - {dataset: nwm30_medium_range_mem1, filter: "hour(reference_time) = 0"}

pair:
  relation: usgs_nwm30_crosswalk

fields:
  - {name: lead_time_bin, fn: lead_time_bins, width: PT6H}
  - {name: high_flow, expr: "primary_value > 1000"}

filter: "value_time >= '2010-10-01' AND value_time < '2020-10-01'"

aggregate:
  - group_by: [primary_series_key, secondary_dataset, lead_time_bin]
    metrics:
      - count
      - relative_bias
      - rmse
      - nash_sutcliffe_efficiency
      - {name: hits, threshold: {value: 1000, unit: "[cft_i]/s"}}
    skill: {reference: nwm30_medium_range_mem1, metrics: [rmse]}
    bootstrap: {method: circular_block, block: P1M, reps: 1000, seed: 42}

output:
  unit: "[cft_i]/s"
  include_pairs: false
```

The `high_flow` expression compares against a bare number; how a unit is written inside an
expression is still open (`spec/expressions.md`).

## Steps

- **`inputs`**: `primary` is one dataset (the reference side); `secondary` is one or more. Each is
  `{dataset, filter?}`; the dataset's name becomes the `secondary_dataset` value. `filter` applies
  before pairing.
- **`pair`**: `relation: <crosswalk>` pairs series by `series_key` through a crosswalk, or
  `on_attribute: <attr>` when both sides share a series attribute. The paired table carries
  `primary_series_key`, `secondary_series_key`, `value_time`, `reference_time`, `primary_value`,
  `secondary_value`, `secondary_dataset`, plus `member` / `quantile` / `period_start` /
  `period_end` when present. Values pair only when their resolved period windows are equal
  (`standards/periods.md`).
- **`fields`**: derived columns, `expr: <expression>` or a named `fn:` (`lead_time_bins`, `within`
  a layer, `attribute` of a location).
- **`filter`**: one row filter after fields, before aggregation.
- **`aggregate`**: one or more blocks of `group_by` + `metrics`. A metric is a name or an object
  with parameters. `skill` scores against a `reference` dataset.
- **`bootstrap`**: optional, on an aggregate block: `method` (`iid`, `circular_block`,
  `stationary`, `year_block`), `block` (ISO 8601 days or months, or rows), `reps`, `seed`,
  `quantiles`; the procedure is pinned in `standards/bootstrap.md`.
- **`output`**: `unit`, the evaluation unit (defaults to the primary's unit; `standards/units.md`),
  `include_pairs`, and geometry/attributes for a geospatial group key.
- **`periods`**: optional; inline definitions of any non-shared period the spec names
  (`standards/periods.md`).

## Dataset references

A reference is a name: lowercase letters, digits and `_`, starting with a letter, at most 60
characters. Used by `dataset`, `relation` and `reference`. `<name>` is a dataset in the
evaluation's project; `public.<name>` is one in the `public` project. (`<project>.<name>` may come
later.)

## Validation (reject before any run)

- An unknown dataset, crosswalk, column, metric, period or unit.
- A `filter` / `expr` that uses a column not present at that step, or anything off the allow-list.
- A `pair.relation` that doesn't connect the two inputs.
- A `group_by` column not present after inputs, pair and fields.
- A unit-bearing parameter without a unit, or with a unit that doesn't convert.

## Versioning

`version` starts at 1 (VeriRace's v1). Breaking changes bump it; a loader supports the current and previous
version so saved specs keep running across one migration window.

## Decided (from VeriRace v1)

The schema starts from VeriRace's `evaluation-spec-v1.schema.json`, with these decided where the
draft and VeriRace v1 differed (`DECISIONS.md`):

| | Decided |
| --- | --- |
| `version` | starts at `1` |
| metric names | one canonical name per metric (teehr's long form, e.g. `nash_sutcliffe_efficiency`), plus frozen aliases (`nse`, `kge`, ...) accepted wherever a spec names a metric; results are always labelled with the canonical name (`standards/metrics.md`) |
| `bootstrap` | on the aggregate stage (VeriRace's shape): `method`, `block`, `reps`, `seed`; methods `iid`, `circular_block`, `stationary`, `year_block` (alias `gumboot`); the procedure in `standards/bootstrap.md` |
| durations | ISO 8601 everywhere (`PT6H`, `PT30M`, `P1D`, `P1M`) |
| window columns on pairs | one `period_start`, `period_end`: values only pair when their windows are equal, so per-side columns would always match |

**Metrics aren't shared yet.** Names, definitions and parameters decide the numbers, so the same
spec can't give the same results until they're agreed. `standards/metrics.md` holds the naming
rules; its entries are seeded from teehr's definitions, then reconciled with VeriRace's registry.

## Open

- Generated secondary inputs (persistence, climatology, blend): VeriRace has them; v0 or later?
- How `member` and `reference_time` flow through pairing for forecast verification.
- A crosswalk whose two sides share a series table (e.g. two NWM forecast ranges): `pair.relation`
  can't tell which side is primary from the inputs, so the spec must say (e.g.
  `relation: {crosswalk: x, primary: left}`).
- Directional crosswalk columns (a `weight` for many-to-one aggregation): which side the weights
  aggregate onto, and how a metric uses them.
- Whether `output` geometry/format belongs in the spec or in API content negotiation.
