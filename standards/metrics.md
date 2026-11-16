# Metrics

- **Status:** naming rules agreed 2026-11-16 (`DECISIONS.md`). Entries to be seeded from teehr's
  metric definitions, then reconciled with VeriRace's registry.

A metric's name, definition and parameters decide its numbers, so both systems must mean the same
thing by each one.

## Names

- **One canonical name per metric**, teehr's long form (`nash_sutcliffe_efficiency`,
  `kling_gupta_efficiency`, `root_mean_squared_error`). Results are always labelled with it, so
  two specs that name the same metric differently give identical outputs.
- **Aliases** (`nse`, `kge`, `rmse`, ...) are accepted wherever a spec names a metric.
  - An alias maps to exactly one metric with fixed default parameters (e.g. which KGE variant).
  - Aliases are frozen: never removed, never repointed.
  - An abbreviation two metrics could claim is given to neither.

## Each entry

| Field | Meaning |
| --- | --- |
| `name` | the canonical name |
| `aliases` | accepted alternatives |
| `definition` | the formula, including edge cases (no values, zero variance) |
| `parameters` | names, units (`{value, unit}` when unit-bearing), defaults |
| `mergeable` | whether it computes from metric state (`standards/metric-state.md`); quantile metrics marked mergeable snap to sketch buckets on every path |
| `unit` | the result's unit: the evaluation unit, its square, or none |

## Entries

(To seed.)
