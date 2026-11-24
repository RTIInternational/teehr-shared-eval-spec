# Changelog

## Unreleased

- `data-api.md`: TEEHR marks layer geometry in Arrow as `geoarrow.wkb`; that known difference is removed.
- `data-api.md`, `units.md`: a `unit` parameter on reads (a timeseries' `value` and run results).
  Results convert by each metric's kind (value, sum, difference, difference², ratio, time,
  invariant), with its options, transform and stage, as VeriRace does; columns that can't be
  converted (an offset under ratios, sums and transforms) give 422 `unit_not_convertible`. With `unit`, a read's `filter` sees values in that unit. The
  response gives its unit in `X-Unit`. New known differences: `unit` on reads, the unit header and
  VeriRace's layer routes.
- `timeseries.md`: optional `quality` becomes `qualifier` (the source's codes, comma-separated) and
  `approval_status` is added (the source's review state, e.g. USGS Provisional/Approved); both may be
  null. `uploads.md` mapping fields renamed to match; the `hydro_metrics` input carries
  `approval_status`.
- `conformance/`: the case format, the first seven cases and their unit table; groups exist only with complete pairs; the repository is public.
- `metrics.md`: `center_of_timing` and `standard_deviation_of_timing` (from teehr; in teehr-core).
- `metrics.md`: `add_epsilon` dropped (replaces the #4 decision); a transform takes an
  `offset` (`{value, unit}`) for zeros under `log` / `reciprocal`. `#8`'s decisions marked accepted.
- `metrics.md`: `proportion_correct`; `flow_duration_curve_slope`'s `log` option and plotting position;
  signatures over complete pairs (from comparing teehr with teehr-core).
- `data-api.md`: TEEHR's `/data` `filter` is now the shared expression language.
- `standards/uploads.md` (agreed): the shared upload contract (steps, kinds, formats, status,
  preview, ingest, mapping fields, result). `data-api.md`: layer geometry exchange (WKB, GeoArrow,
  GeoParquet, GeoJSON, `bbox`), the shape of `errors` entries, a first shared `code`.
- `standards/bootstrap.md` (agreed): the bootstrap procedure pinned, from VeriRace's
  implementation: units, per-group seeds and streams, block draws, intervals; `gumboot` an alias
  of `year_block`; conformance cases for draws and for given draws.
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

- `standards/metrics.md` (draft): canonical metric names with frozen aliases, definitions, parameters and which merge from state; open items listed under "To decide", with TEEHR's proposed calls.
- Seeded from the TEEHR design docs agreed with VeriRace so far (shared contract v0, spec schema
  v0, ADR-0012 periods, data model v1).
