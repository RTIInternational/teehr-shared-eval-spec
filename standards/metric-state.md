# Mergeable metric state

- **Status:** agreed 2026-11-13 (shared contract v0, section A). `contract_version: 0`.

Each engine writes its own producer and consumer; this fixes only the format.

**Principle:** any slice of a group's pairs yields a state row; rows merge by addition (with a
mean correction for centered moments); a metric computes identically from raw pairs, from merged
state, or from a bootstrap drawn from state. The exact columns come from the metric registry
(each metric declares the components it needs), so agreeing the registry plus this encoding fixes
the columns.

**Row identity:** `group_id` (plus any partition or shard key the engine merges away),
`contract_version`, and `n` (valid-pair count). Three column families:

## 1. `components`: centered moments

Additive by the Chan parallel merge; bit-exact.

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

The `<threshold>` in a key is written in the threshold's canonical SI value so keys from both
systems match. *(Proposed; follows from units on thresholds, not yet agreed.)*

## 3. `~primary` / `~secondary`: quantile sketches (DDSketch)

For order-statistic metrics. DDSketch merges by adding bucket counts, exactly and in any order,
which the path-independence invariant needs (t-digest was rejected: its merge is approximate and
order-dependent).

- **Relative accuracy** α = 0.001, so γ = (1+α)/(1−α) ≈ 1.002002. A positive `x` maps to bucket
  `i = ⌈log_γ(x)⌉`; a bucket's estimate is `2·γ^i/(γ+1)`, within 0.1% relative error.
- **Zero / sub-floor:** a `zero` counter; magnitudes below the configured `min` fold into it.
- **Signed values:** a sign-mirrored bucket set indexed on `|x|` for the negative side.
- **Bounded size:** a fixed index range from configured `[min, max]`; counts outside collapse into
  the boundary buckets. γ, the range and the collapsing are pinned by `contract_version`.
- Stored as a fixed-length `int64` array in index order. Merge = add counts per index.

## Encoding and invariants

- `float64` for moments, `int64` for counts, each sketch a fixed-length `int64` array. `NaN` where
  a metric is undefined; null = absent component.
- **Path independence** (conformance cases): `pairs == merged state == bootstrap from state`
  exactly for all three families. A sketch quantile is additionally within 0.1% of the true value.

## Open

- The sketch `[min, max]` range per variable (pinned by `contract_version`).
