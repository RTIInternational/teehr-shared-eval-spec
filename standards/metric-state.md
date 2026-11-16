# Mergeable metric state

- **Status:** agreed 2026-11-13 (shared contract v0, section A). `contract_version: 0`.

Each engine writes its own producer and consumer, and stores state its own way: state rows don't
move between systems. This fixes what determines the numbers: the components, the merge laws, the
sketch mapping, the zero floor, snapping on every path, and the quantile definition. The column
names and encodings below are descriptive; an engine may store the same information differently
(VeriRace keeps plain and centered sums, and sparse sketches; see "Known differences").

**Principle:** any slice of a group's pairs yields a state row; rows merge by addition (with a
mean correction for centered moments); a metric computes identically from raw pairs, from merged
state, or from a bootstrap drawn from state. The exact columns come from the metric registry
(each metric declares the components it needs), so agreeing the registry plus this encoding fixes
the columns.

**Row identity:** `group_id` (plus any partition or shard key the engine merges away),
`contract_version`, and `n` (valid-pair count). Three column families:

## 1. `components`: centered moments

Additive by the Chan parallel merge.

| Column | Meaning |
| --- | --- |
| `mean_primary`, `mean_secondary` | running means |
| `M2_primary`, `M2_secondary` | centered sum of squares |
| `C_primary_secondary` | centered co-moment |
| `mean_error`, `M2_error` | error = `secondary − primary` |
| `sum_abs_error` | for MAE (plain additive) |
| `min_primary`, `max_primary`, `min_secondary`, `max_secondary` | extremes |

Merge parts A and B (δ = `mean_B − mean_A`, `n = n_A + n_B`):

```
mean = mean_A + δ · n_B / n
M2   = M2_A + M2_B + δ²  · n_A · n_B / n
C    = C_A  + C_B  + δ_primary · δ_secondary · n_A · n_B / n
```

Extremes merge by min/max; `n` and `sum_abs_error` by addition. These give exactly: bias,
MSE/RMSE, MAE, variance/std, covariance, Pearson r, NSE, KGE.

## 2. `+<key>`: additive sums keyed by metric and parameters

Plain addition, e.g. contingency counts `+hits@<threshold>`, `+misses@<threshold>`,
`+false_alarms@<threshold>`, `+correct_negatives@<threshold>`, and per-case score sums. `int64` for
counts, `float64` for score sums.

Keys are engine-internal: state isn't exchanged, so they needn't match across systems.

## 3. `~primary` / `~secondary`: quantile sketches (DDSketch)

For order-statistic metrics. DDSketch merges by adding bucket counts, exactly and in any order,
which the path-independence invariant needs (t-digest was rejected: its merge is approximate and
order-dependent).

- **Relative accuracy** α = 0.001, so γ = (1+α)/(1−α) ≈ 1.002002. A positive `x` maps to bucket
  `i = ⌈log_γ(x)⌉`; a bucket's estimate is `2·γ^i/(γ+1)`, within 0.1% relative error.
- **Sketched in the evaluation unit.** Values are sketched after conversion to the evaluation
  unit (`output.unit`). Buckets are logarithmic, so the same data sketched in another unit snaps
  to different representatives; sketching in one unit keeps paths and systems equal.
- **Zero / sub-floor:** a `zero` counter; magnitudes below `1e-12` (in the evaluation unit) fold
  into it.
- **Signed values:** a sign-mirrored bucket set indexed on `|x|` for the negative side.
- **No range collapsing.** The index is unbounded (store it sparsely); collapsing counts into
  boundary buckets would void the 0.1% guarantee for values outside the range.
- Merge = add counts per bucket.
- **Snapping on every path.** A quantile metric takes its quantile from bucket representatives on
  every path, including from raw pairs, so pairs, merged state and bootstrap agree.
- **Quantile definition.** With `n` values (counts, or bootstrap weights) and level `q`: let
  `t = q·n`; `lo` is the first bucket whose cumulative count reaches `t`, `hi` the first whose
  cumulative count exceeds `t` (each capped at the last bucket). The quantile is the mean of their
  representatives; NaN without values. Another definition (e.g. linear interpolation) gives other
  numbers.

## Encoding and invariants

- `float64` for moments, `int64` for counts. `NaN` where a metric is undefined; null = absent
  component.
- **Path independence** (conformance cases): `pairs == merged state == bootstrap from state`.
  Counts and sketch quantiles are exact; floating-point moments agree within the conformance
  tolerance, since merge order changes their rounding. A sketch quantile is additionally within
  0.1% of the true value.

## Known differences (not shared)

| | TEEHR | VeriRace |
| --- | --- | --- |
| Moments | Chan means and `M2` | plain sums and centered sums (`ss_*`, `sp_*`) |
| Sketch storage | (TEEHR's choice) | sparse list of `{bucket, weight}`, `float64` weights |
