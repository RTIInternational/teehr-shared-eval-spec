# Conformance cases

*Proposed layout; no cases yet.*

Cases both systems run in CI. A disagreement between systems shows up as a failing case, not an
email thread.

## Layout

```
cases/<name>/
  case.yaml        what it checks, and the contract_version it was added in
  spec.yaml        the evaluation spec (or invalid-spec.yaml for a rejection case)
  inputs/          datasets as Parquet, plus their dataset-level metadata
  expected/        results as Parquet, or expected-error.json for a rejection case
```

## Comparing results

- Counts and `components` metrics: exact.
- Sketch quantiles: identical across paths, and within 0.1% of the true value.
- Other floating-point results: within `tolerance` from `case.yaml` (relative), default 1e-12.

## First cases to add

- path independence: pairs vs merged state vs bootstrap-from-state, each metric family;
- unit conversion: the same threshold in `[cft_i]/s` and `m3/s` gives identical counts;
- a bare-number threshold is rejected;
- period pairing: `hour_starting` vs `hour_ending` don't pair; calendar spans across Jan 1;
- an off-list expression is rejected, including a bare `log`;
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
