# Units

- **Status:** agreed 2026-11-15.

## Rules

- **UCUM symbols** everywhere a unit is named (e.g. `[cft_i]/s`, `m3/s`, `mm`, `Cel`).
- **Every unit-bearing spec parameter is `{value, unit}`**: contingency thresholds, event and ramp
  thresholds, and (once the syntax exists, `spec/expressions.md`) numeric values compared against
  unit-bearing columns in filters.
  ```yaml
  threshold: {value: 1000, unit: "[cft_i]/s"}
  ```
- **Unitless:** quantile thresholds and thresholds that reference a column.
- **Bare numbers are refused.** A unit-bearing parameter without a unit fails validation. Existing
  specs are converted, not grandfathered.
- **Conversion is affine:** `si = value × to_si + offset`. A unit that doesn't convert to the
  dimension of the value it's compared with fails validation.
- **`output.unit`** is the unit for plain thresholds and display; it defaults to the primary
  dataset's unit.

## Known differences (not shared)

| | TEEHR | VeriRace |
| --- | --- | --- |
| Stored unit | the variable's canonical unit, converted on write | as uploaded, converted at pairing |
| Variables | its own registry, each with a canonical unit | free text |

Specs don't depend on either, because every unit-bearing parameter carries its unit.
