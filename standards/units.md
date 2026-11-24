# Units

- **Status:** agreed 2026-11-15.

## Rules

- **UCUM symbols** everywhere a unit is named (e.g. `[cft_i]/s`, `m3/s`, `mm`, `Cel`).
- **Every unit-bearing spec parameter is `{value, unit}`**: contingency thresholds, event and ramp
  thresholds, a numeric `normalize_by` (e.g. installed capacity), a transform's `offset`, and (once the syntax exists,
  `spec/expressions.md`) numeric values compared against unit-bearing columns in filters.
  ```yaml
  threshold: {value: 1000, unit: "[cft_i]/s"}
  ```
- **Unitless:** quantile thresholds and thresholds that reference a column.
- **Bare numbers are refused.** A unit-bearing parameter without a unit fails validation. Existing
  specs are converted, not grandfathered.
- **Conversion is affine:** `si = value × to_si + offset`. A unit that doesn't convert to the
  dimension of the value it's compared with fails validation.
- **Differences and magnitudes convert by scale alone.** A threshold on changes (a ramp, scored on
  change fields), a normalizer and a transform's `offset` don't take the offset: a 5 K ramp is a 5 Cel ramp, not
  −268.15 Cel. A threshold on values does: 300 K is 26.85 Cel.
- **`output.unit`** is the evaluation unit: both sides' values are converted to it at pairing, so
  pair-level fields, filters and metrics are in it, and every `{value, unit}` parameter is converted
  to it. It defaults to the primary dataset's unit.

## Converting on read

A read's `unit` (`data-api.md`) converts what it returns, by `x' = a·x + b` from the stored unit
(results: the evaluation unit) to `unit`; `b` is nonzero only between scales with different zero
points (e.g. `Cel` and `[degF]`). The unit must be of the values' dimension, else 422 (`errors`
entry `loc` `["query", "unit"]`). These are VeriRace's rules (`verirace.metrics.units`).

- **A timeseries' `value`:** `a·x + b`.
- **A result column:** by its metric's kind:

  | Kind | Converts | Metrics |
  | --- | --- | --- |
  | value | `a·x + b` | `minimum`, `maximum`, `average` |
  | sum | `a·x`; not convertible when `b ≠ 0` | `sum` |
  | difference | `a·x` | `mean_error`, `mean_absolute_error`, `root_mean_square_error`, `max_value_delta`, `mean_crps_ensemble`, `mean_pinball_loss`, `mean_interval_width`, `mean_interval_score`, `ensemble_spread`, `ensemble_mean_root_mean_square_error` |
  | difference² | `a²·x` | `mean_square_error`, `variance` |
  | ratio | unchanged; not convertible when `b ≠ 0` | `relative_bias`, `mean_absolute_relative_error`, `multiplicative_bias`, `relative_median`, `relative_minimum`, `relative_maximum`, `mean_absolute_percentage_error`, `symmetric_mean_absolute_percentage_error`, `normalized_mean_absolute_error`, `normalized_root_mean_square_error`, `coefficient_of_variation_ratio`, `kling_gupta_efficiency`, `kling_gupta_efficiency_mod1`, `annual_peak_flow_bias`, `center_of_timing`, `standard_deviation_of_timing`, `flow_duration_curve_slope` |
  | time | unchanged | `max_value_time_delta`, `max_value_time` |
  | invariant | unchanged | every other metric (counts, correlations, efficiencies, contingency, Brier, coverage, rank metrics) |

  Options and transforms change it:

  - `normalized_mean_absolute_error` and `normalized_root_mean_square_error` with
    `normalize_by: range_primary` are invariant; `flow_duration_curve_slope` with `log: false` is a
    difference.
  - With a transform, nothing converts when `b ≠ 0` (except time and invariant metrics). Otherwise:
    `log` adds `ln a` to a value, leaves differences and ratios unchanged, and can't convert a sum;
    `exp` converts only when `a = 1`; a power `p` (`sqrt` ½, `square` 2, `cube` 3, `reciprocal` −1,
    `abs` 1) leaves ratios unchanged and multiplies the rest by `aᵖ` (`a²ᵖ` for difference²).
  - A metric of change fields (a ramp): a value or sum becomes a difference. A metric of other
    columns than the paired values: not convertible.
  - Bootstrap interval columns convert as their metric; skill-score columns are unchanged.
  - Later stages: `count` is unchanged; `mean`, `median`, `min` and `max` convert as their column;
    `std` by `|s|` of it; `sum` by `s` alone, only when its column has no offset.
- **Not convertible:** 422 with `code` `unit_not_convertible`, naming the columns in `detail`.
  Select the other columns (`columns`), or evaluate in that unit (`output.unit`).

## Known differences (not shared)

| | TEEHR | VeriRace |
| --- | --- | --- |
| Stored unit | the variable's canonical unit, converted on write | as uploaded, converted at pairing |
| Variables | its own registry, each with a canonical unit | its own registry, each with a canonical unit; values stay in the uploaded unit, which must convert to it |

Specs don't depend on either, because every unit-bearing parameter carries its unit.
