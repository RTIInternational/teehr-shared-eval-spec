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
  input names the dataset directly. A `filter` narrows within it by time, location, member or
  quality, never by variable.
- **Validated before any run** against the system's datasets, columns, metrics and units.
- **Units are explicit** on every unit-bearing parameter (`standards/units.md`).
- **Versioned.** `version` is recorded on every run.

## Steps (always in this order)

`inputs → pair → fields → filter → aggregate → output`

## Example

```yaml
version: 0
name: nwm30-vs-usgs-by-lead-time

inputs:
  primary: {dataset: usgs_streamflow}
  secondary:
    - {dataset: nwm30_short_range}
    - {dataset: nwm30_medium_range_mem1, filter: "hour(reference_time) = 0"}

pair:
  relation: usgs_nwm30_crosswalk

fields:
  - {name: lead_time_bin, fn: lead_time_bins, width: 6h}
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

bootstrap: {method: block, block: P1M, iterations: 1000, seed: 42, ci_level: 0.90}

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
- **`bootstrap`**: optional; `method: block` resamples metric-state chunks
  (`standards/metric-state.md`).
- **`output`**: `unit` for plain thresholds and display (defaults to the primary's unit),
  `include_pairs`, and geometry/attributes for a geospatial group key.
- **`periods`**: optional; inline definitions of any non-shared period the spec names
  (`standards/periods.md`).

## Dataset references

A reference is a name: lowercase letters, digits and `_`, starting with a letter, at most 60
characters. Used by `dataset`, `relation` and `reference`.

## Validation (reject before any run)

- An unknown dataset, crosswalk, column, metric, period or unit.
- A `filter` / `expr` that uses a column not present at that step, or anything off the allow-list.
- A `pair.relation` that doesn't connect the two inputs.
- A `group_by` column not present after inputs, pair and fields.
- A unit-bearing parameter without a unit, or with a unit that doesn't convert.

## Versioning

`version` starts at 0. Breaking changes bump it; a loader supports the current and previous
version so saved specs keep running across one migration window.

## Open

- How a reference names a dataset in another project (TEEHR uses `public.` / `platform.`
  prefixes for its system projects).
- Generated secondary inputs (persistence, climatology, blend): VeriRace has them; v0 or later?
- How `member` and `reference_time` flow through pairing for forecast verification.
- Whether `output` geometry/format belongs in the spec or in API content negotiation.
