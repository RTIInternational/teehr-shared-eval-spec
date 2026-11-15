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
- an off-list expression is rejected.
