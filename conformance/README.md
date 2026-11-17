# Conformance cases

- **Status:** first cases added 2026-11-17. Run by teehr-core's CI with its engine (`python -m
  teehr_core.conformance conformance/cases`); a system runs them with its own `evaluate` through
  the same runner (`teehr_core.conformance.run_cases`).

Cases both systems run in CI. A disagreement between systems shows up as a failing case, not an
email thread.

## Layout

```
cases/<name>/
  case.yaml        description, added (date); optional tolerance, keys
  spec.yaml        the evaluation spec (or invalid-spec.yaml for a rejection case)
  inputs/          datasets.yaml and its Parquet files
  expected/        results.parquet, or error.yaml for a rejection case
units.csv          the unit table every case uses (UCUM; aliases separated by |)
```

Periods are `standards/periods.csv`.

`inputs/datasets.yaml` lists each dataset under `datasets` with `role` (`observed`, `simulated`,
`forecast`, `auxiliary`), `file`, and as applicable `series_file`, `variable`, `unit`, `statistic`,
`period`, `kind` (`crosswalk`). The files follow `standards/timeseries.md` (and the data model's
series, layer and crosswalk shapes).

`expected/results.parquet` is the final stage's table. `expected/error.yaml` gives `about`: a word
the refusal's message must contain (messages differ between systems; the refusal doesn't).

## Comparing results

- Rows are matched on `keys` (default: the stages' `group_by` columns); row order doesn't matter.
  The columns must be the same.
- Null and NaN are alike; integers, strings and times compare exactly.
- Floating-point results: within the relative `tolerance` from `case.yaml`, default 1e-12.
- Sketch quantiles: identical across paths, and within 0.1% of the true value.

## Cases

| Case | Checks |
| --- | --- |
| `hydro_metrics`, `energy_metrics` | every metric the fixture specs ask for, on real data with gaps, through a crosswalk, with units converted at pairing |
| `undefined_results` | constant observations and a single pair give null NSE, KGE and Pearson; missing secondary values leave their pairs out |
| `empty_series` | a group exists only if it has at least one complete pair: a series whose secondary values are all missing gets no row |
| `threshold_units` | the same threshold in `m3/s` and `[cft_i]/s` gives the same counts |
| `reject_bare_threshold` | a threshold without a unit is refused |
| `reject_bare_log` | one-argument `log()` is refused |

## Cases to add

- path independence: pairs vs merged state vs bootstrap-from-state, each metric family;
- period pairing: `hour_starting` vs `hour_ending` don't pair; calendar spans across Jan 1;
- an off-list expression is rejected (a bare `log` is done);
- `ln` and `log10` give the same values on both engines;
- a ramp threshold given in K and in Cel scores the same events (scale-only conversion);
- sketch quantiles at levels that fall between and on bucket boundaries (the quantile definition);
- bootstrap draws: for a seed, group key, method, block, `n` and `reps`, the exact blocks
  (`standards/bootstrap.md`; tests the random streams alone, no metrics);
- bootstrap from given draws: both systems take the same unit weights and give identical intervals
  (tests the metrics and quantiles without the random streams);
- `year_block` with water years across a partial year, and a group under `min_years`;
- a `stationary` replicate that needs extra draws from `more`;
- day and month blocks: a bootstrap from state equals one from pairs.
