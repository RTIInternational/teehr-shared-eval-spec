# Changelog

## Unreleased

- VeriRace review: `output.unit` is the evaluation unit; `normalize_by` carries a unit; ramp
  thresholds and normalizers convert by scale alone; `log` replaced by `ln`/`log10`; interim units
  in expressions by step; metric state fixes the math (quantile definition, snapping on every path,
  zero floor, no range collapsing) rather than the storage; known differences recorded for the data
  API, metric state and periods; spec differences from VeriRace v1 listed to settle.

- Seeded from the TEEHR design docs agreed with VeriRace so far (shared contract v0, spec schema
  v0, ADR-0012 periods, data model v1).
