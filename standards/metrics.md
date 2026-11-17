# Metrics: names, aliases and definitions

- **Status:** draft 2026-11-16, for agreement. Seeded from TEEHR's metric names; definitions and
  parameters from VeriRace's registry, which has the larger set. Where the two systems define a
  name differently, it's listed under "To decide".

## Naming

- Each metric has one **canonical name**, in full words (`kling_gupta_efficiency`). Results are
  always labelled with it.
- A metric may have **aliases** (`kge`), accepted wherever a spec names a metric. An alias maps to
  exactly one metric with its default parameters, is never reused for another metric, and is
  never removed. An abbreviation two metrics could claim isn't given to either.
- A name or alias is lowercase words joined by `_`.
- **Merges from state:** the metric is computed from mergeable state
  (`standards/metric-state.md`), so it's the same from pairs, merged state and a bootstrap
  drawn from state. Quantiles in such metrics are snapped (within 0.1%) on every path. A metric
  that doesn't merge is computed from the pairs and stays exact.

`obs` is the primary value, `pred` the secondary. Means are over pairs; `std` is the
population standard deviation.

**`transform`** (default none) is applied to both sides before the metric:

| Transform | Aliases | Value | Undefined for |
| --- | --- | --- | --- |
| `log` | | natural log | ≤ 0 |
| `sqrt` | | square root | < 0 |
| `square` | | x² | |
| `cube` | | x³ | |
| `exp` | | eˣ | overflow |
| `reciprocal` | `inv` | 1 / x | 0 |
| `abs` | | \|x\| | |

- A pair whose transformed value is undefined (or not finite) is left out of that metric.
- **No transform** on the four contingency counts, the six scores from them, `mean_brier_score`,
  and the ensemble and rank metrics (`ensemble_spread`, `ensemble_mean_root_mean_square_error`,
  `spread_skill_ratio`, `rank_frequency`, `rank_outlier_fraction`, `rank_histogram_reliability`):
  it's meaningless there (an increasing transform doesn't change which values are events;
  `reciprocal` flips them). A spec that asks for one is refused.

**`add_epsilon`** (default false) adds `1e-6`, in the evaluation unit (`output.unit`), to every
denominator, and to both sides before `log` and `reciprocal`, so a zero gives a large finite value
instead of an undefined one. The spec fixes the evaluation unit, so the result is the same on both
systems.

**Result units:** a metric of values (`mean_error`, `mean_absolute_error`,
`root_mean_square_error`, `max_value_delta`, `minimum`, `maximum`, `average`, `sum`,
`ensemble_spread`, `mean_crps_ensemble`, interval widths and scores) is in the evaluation unit
(`output.unit`); `mean_square_error` and `variance` in its square; `max_value_time_delta` in
seconds; counts, ratios, scores and correlations have none.

## Deterministic

| Canonical | Aliases | Definition | Parameters | Merges |
| --- | --- | --- | --- | --- |
| `count` | | number of pairs | | yes |
| `mean_error` | `me` | mean(pred − obs) | | yes |
| `mean_absolute_error` | `mae` | mean(\|pred − obs\|) | | yes |
| `mean_square_error` | `mse`, `mean_squared_error` | mean((pred − obs)²) | | yes |
| `root_mean_square_error` | `rmse` | sqrt(mean((pred − obs)²)) | | yes |
| `max_value_delta` | | max(pred) − max(obs) | | yes |
| `relative_bias` | `relbias` | sum(pred − obs) / sum(obs) | | yes |
| `mean_absolute_relative_error` | | sum(\|pred − obs\|) / sum(obs) | | yes |
| `multiplicative_bias` | `multbias`, `relative_mean`, `relmean` | mean(pred) / mean(obs) | | yes |
| `relative_median` | | median(pred) / median(obs) | | yes (sketch) |
| `relative_minimum` | `relative_min` | min(pred) / min(obs) | | yes |
| `relative_maximum` | `relative_max` | max(pred) / max(obs) | | yes |
| `relative_standard_deviation` | `relative_std`, `relstd`, `variability_ratio`, `vr` | std(pred) / std(obs) | | yes |
| `mean_absolute_percentage_error` | `mape` | 100 · mean(\|pred − obs\| / \|obs\|), over obs ≠ 0 | | yes |
| `symmetric_mean_absolute_percentage_error` | `smape` | 100 · mean(2\|pred − obs\| / (\|obs\| + \|pred\|)) | | yes |
| `normalized_mean_absolute_error` | `nmae` | MAE / normalizer | `normalize_by`: `mean_primary` (default), `range_primary`, `{value, unit}`, `{column}` | yes |
| `normalized_root_mean_square_error` | `nrmse` | RMSE / normalizer | as above | yes |
| `mean_absolute_scaled_error` | `mase` | mean(\|pred − obs\| / scale), scale a stored in-sample naive error | `scale` (a pair column) | yes |
| `pearson_correlation` | `pearson_r` | Pearson correlation | | yes |
| `spearman_correlation` | `spearman_r` | Spearman rank correlation (ties averaged) | | no |
| `r_squared` | | pearson_correlation² | | yes |
| `root_mean_standard_deviation_ratio` | `rsr` | RMSE / std(obs) | | yes |
| `nash_sutcliffe_efficiency` | `nse` | 1 − sum((pred − obs)²) / sum((obs − mean(obs))²) | | yes |
| `nash_sutcliffe_efficiency_normalized` | `nnse` | 1 / (2 − NSE) | | yes |
| `kling_gupta_efficiency` | `kge` | 1 − sqrt(sr·(r−1)² + sa·(α−1)² + sb·(β−1)²), α = std ratio, β = mean ratio (Gupta et al. 2009); NaN if std(pred) or std(obs) is 0, whatever the weights | `sr`, `sa`, `sb`: weights on the three terms (default 1 each) | yes |
| `kling_gupta_efficiency_mod1` | `kge_2012`, `kge_mod1` | as KGE with α the ratio of coefficients of variation (Kling et al. 2012) | `sr`, `sa`, `sb`: weights on the three terms (default 1 each) | yes |
| `kling_gupta_efficiency_mod2` | `kge_2021`, `kge_mod2` | 1 − sqrt(sr·(r−1)² + sa·(α−1)² + sb·β²), β = (mean(pred) − mean(obs)) / std(obs) (Tang et al. 2021) | `sr`, `sa`, `sb`: weights on the three terms (default 1 each) | yes |
| `max_value_time_delta` | | time of max(pred) − time of max(obs), in seconds | | yes |
| `annual_peak_flow_bias` | `annual_peak_relative_bias` | sum over years of (annual max(pred) − annual max(obs)) / sum of annual max(obs) | `year_start_month` (default 1, calendar years; 10 for US water years) | no |

## Categorical (an event is a value ≥ `threshold`)

All take `threshold` (`{value, unit}`, `{quantile}` or `{column}`) and merge from state.

| Canonical | Aliases | Definition |
| --- | --- | --- |
| `hits` | | obs and pred both events |
| `misses` | | obs an event, pred not |
| `false_alarms` | | pred an event, obs not |
| `correct_negatives` | | neither |
| `probability_of_detection` | `pod` | hits / (hits + misses) |
| `probability_of_false_detection` | `pofd` | false_alarms / (false_alarms + correct_negatives) |
| `false_alarm_ratio` | `far` | false_alarms / (hits + false_alarms) |
| `critical_success_index` | `csi` | hits / (hits + misses + false_alarms) |
| `success_ratio` | | hits / (hits + false_alarms) |
| `frequency_bias_index` | `frequency_bias` | (hits + false_alarms) / (hits + misses) |
| `proportion_correct` | `accuracy` | (hits + correct_negatives) / count |

## Signatures (of one side: `side`, default primary)

A signature is computed over the group's complete pairs (both sides present), the same sample
every other metric uses, not over all of that side's own values.

| Canonical | Aliases | Definition | Merges |
| --- | --- | --- | --- |
| `minimum` | `min_value` | min | yes |
| `maximum` | `max_value` | max | yes |
| `average` | `mean_value` | mean | yes |
| `sum` | `sum_value` | sum | yes |
| `variance` | | population variance | yes |
| `max_value_time` | | time of the max (earliest if repeated) | yes |
| `flow_duration_curve_slope` | `fdc_slope` | (ln Q(e1) − ln Q(e2)) / (e2 − e1), `exceedance` [e1, e2] default [0.33, 0.66] (Sawicz et al. 2011); with `log: false`, (Q(e1) − Q(e2)) / (e2 − e1). Q(e) is the value exceeded a fraction e of the time: the quantile at level 1 − e by `metric-state.md`'s quantile definition, snapped to sketch buckets | yes (sketch) |

## Probabilistic

| Canonical | Aliases | Definition | Parameters | Merges |
| --- | --- | --- | --- | --- |
| `mean_crps_ensemble` | `crps` | CRPS of ensemble forecasts, averaged over cases | `estimator`: `standard`, `fair` | yes |
| `mean_brier_score` | `brier_score` | mean of (p − o)², p the members' share ≥ threshold | `threshold` | yes |
| `mean_pinball_loss` | `pinball_loss` | mean quantile loss over quantile forecasts | | yes |
| `interval_coverage` | | share of cases with obs inside the central interval | `interval` (two levels, default [0.05, 0.95]) | yes |
| `mean_interval_width` | `interval_width` | mean width of the central interval | `interval` | yes |
| `mean_interval_score` | `winkler_score` | width + (2/α)·distance outside, averaged | `interval` | yes |
| `ensemble_spread` | | sqrt(mean over cases of the members' variance) | | yes |
| `ensemble_mean_root_mean_square_error` | `ensemble_mean_rmse` | RMSE of the ensemble mean | | yes |
| `spread_skill_ratio` | | spread / ensemble-mean RMSE, with the (m+1)/m correction | | yes |
| `rank_frequency` | | share of cases with obs at rank `rank` among members | `rank` | yes |
| `rank_outlier_fraction` | | share of cases with obs outside the members' range | | yes |
| `rank_histogram_reliability` | | sum over ranks of \|frequency − 1/(m+1)\| | | no |

## Decided

Recorded in `DECISIONS.md` (2026-11-16): the canonical names and aliases above, including
`variability_ratio` → `relative_standard_deviation` (VeriRace's CV-ratio metric becomes
`coefficient_of_variation_ratio`) and `relative_mean` → `multiplicative_bias`; the four counts in
place of `confusion_matrix`; `max_value_time_delta` in seconds; `year_start_month` (default 1) for
`annual_peak_flow_bias`; the KGE weights; the transforms and `add_epsilon` as above; result units;
events are values `≥` the threshold; CRPS's default estimator `standard`.

## Open

- `center_of_timing`, `standard_deviation_of_timing` (teehr): defined here when VeriRace adds them.
