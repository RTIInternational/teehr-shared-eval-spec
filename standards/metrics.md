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

## Deterministic

| Canonical | Aliases | Definition | Parameters | Merges |
| --- | --- | --- | --- | --- |
| `count` | | number of pairs | | yes |
| `mean_error` | `me` | mean(pred − obs) | | yes |
| `mean_absolute_error` | `mae` | mean(\|pred − obs\|) | | yes |
| `mean_square_error` | `mse`, `mean_squared_error` | mean((pred − obs)²) | | yes |
| `root_mean_square_error` | `rmse` | sqrt(mean((pred − obs)²)) | | yes |
| `max_value_delta` | | max(pred) − max(obs) | | yes |
| `relative_bias` | | sum(pred − obs) / sum(obs) | | yes |
| `mean_absolute_relative_error` | | sum(\|pred − obs\|) / sum(obs) | | yes |
| `multiplicative_bias` | | mean(pred) / mean(obs) | | yes |
| `relative_median` | | median(pred) / median(obs) | | yes (sketch) |
| `relative_minimum` | `relative_min` | min(pred) / min(obs) | | yes |
| `relative_maximum` | `relative_max` | max(pred) / max(obs) | | yes |
| `relative_standard_deviation` | `relative_std` | std(pred) / std(obs) | | yes |
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
| `kling_gupta_efficiency` | `kge` | 1 − sqrt((r−1)² + (α−1)² + (β−1)²), α = std ratio, β = mean ratio (Gupta et al. 2009) | | yes |
| `kling_gupta_efficiency_mod1` | `kge_2012` | as KGE with α the ratio of coefficients of variation (Kling et al. 2012) | | yes |
| `kling_gupta_efficiency_mod2` | `kge_2021` | 1 − sqrt((r−1)² + (α−1)² + β²), β = (mean(pred) − mean(obs)) / std(obs) (Tang et al. 2021) | | yes |
| `max_value_time_delta` | | time of max(pred) − time of max(obs), in hours | | yes |
| `annual_peak_flow_bias` | `annual_peak_relative_bias` | sum over years of (annual max(pred) − annual max(obs)) / sum of annual max(obs) | `water_year_month` (default 1) | no |

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

## Signatures (of one side: `side`, default primary)

| Canonical | Aliases | Definition | Merges |
| --- | --- | --- | --- |
| `minimum` | `min_value` | min | yes |
| `maximum` | `max_value` | max | yes |
| `average` | `mean_value` | mean | yes |
| `sum` | `sum_value` | sum | yes |
| `variance` | | population variance | yes |
| `max_value_time` | | time of the max (earliest if repeated) | yes |
| `flow_duration_curve_slope` | `fdc_slope` | (ln Q(e1) − ln Q(e2)) / (e2 − e1), `exceedance` [e1, e2] default [0.33, 0.66] | yes (sketch) |

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

## To decide

- **`variability_ratio`:** TEEHR defines it as std(pred) / std(obs), the same as its
  `relative_standard_deviation`; VeriRace as the ratio of coefficients of variation (KGE 2012's
  γ). Proposed: drop the name, since `relative_standard_deviation` covers the first; if the CV
  ratio is wanted, name it `coefficient_of_variation_ratio`.
- **`relative_mean`** (TEEHR) is mean(pred) / mean(obs), the same as `multiplicative_bias`.
  Proposed: make it an alias of `multiplicative_bias`.
- **`max_value_time_delta`:** VeriRace reports hours; TEEHR a time delta. Proposed: hours, as a
  number, so it averages and bootstraps like any other metric.
- **`confusion_matrix`** (TEEHR, one output) and VeriRace's four counts: proposed, the four counts
  as metrics; a confusion matrix is those four.
- **TEEHR's `center_of_timing` and `standard_deviation_of_timing`** aren't in VeriRace yet; listed
  for completeness once defined here.
- **Defaults that change numbers:** `annual_peak_flow_bias`'s `water_year_month` (VeriRace 1);
  CRPS's estimator (VeriRace `standard`); the event rule (≥, not >).
