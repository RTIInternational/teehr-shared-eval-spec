# Changelog

## Unreleased

- VeriRace review: `output.unit` is the evaluation unit; `normalize_by` carries a unit; ramp
  thresholds and normalizers convert by scale alone; `log` replaced by `ln`/`log10`; interim units
  in expressions by step; metric state fixes the math (quantile definition, snapping on every path,
  zero floor, no range collapsing) rather than the storage; known differences recorded for the data
  API and metric state; spec differences from VeriRace v1 listed to settle.
- TEEHR review: an input `filter`'s literals are read in the variable's canonical unit; sketches
  are built in the evaluation unit; the data API's known differences gain a TEEHR column; dataset
  references are `<name>` (the evaluation's project) and `public.<name>`; the per-series period
  is no longer a difference.
- TEEHR's calls, accepted by VeriRace: canonical metric names with aliases
  (`standards/metrics.md`); `version` starts at 1; ISO 8601 durations; bootstrap on the aggregate
  stage; one pair of window columns; snapping for mergeable metrics; allow-list additions. Adds
  `DECISIONS.md` and the authorship conventions in `GOVERNANCE.md`.

- Seeded from the TEEHR design docs agreed with VeriRace so far (shared contract v0, spec schema
  v0, ADR-0012 periods, data model v1).
