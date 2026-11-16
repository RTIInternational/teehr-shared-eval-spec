# Units

- **Status:** agreed 2026-11-15.

## Rules

- **UCUM symbols** everywhere a unit is named (e.g. `[cft_i]/s`, `m3/s`, `mm`, `Cel`).
- **Every unit-bearing spec parameter is `{value, unit}`**: contingency thresholds, event and ramp
  thresholds, a numeric `normalize_by` (e.g. installed capacity), and (once the syntax exists,
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
  change fields) and a normalizer don't take the offset: a 5 K ramp is a 5 Cel ramp, not
  −268.15 Cel. A threshold on values does: 300 K is 26.85 Cel.
- **`output.unit`** is the evaluation unit: both sides' values are converted to it at pairing, so
  pair-level fields, filters and metrics are in it, and every `{value, unit}` parameter is converted
  to it. It defaults to the primary dataset's unit.

## Known differences (not shared)

| | TEEHR | VeriRace |
| --- | --- | --- |
| Stored unit | the variable's canonical unit, converted on write | as uploaded, converted at pairing |
| Variables | its own registry, each with a canonical unit | free text (a registry with canonical units is planned) |

Specs don't depend on either, because every unit-bearing parameter carries its unit.
