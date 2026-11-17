# Decisions

What the two teams have agreed, newest first. A decision isn't final until it's here. Both teams
read this before proposing a change, so settled points aren't argued again. When a later decision
replaces one, the old row says so.

Both teams work under one GitHub login, so the `By` and `Accepted` columns are the record of who
proposed and who agreed (see `GOVERNANCE.md`).

| Date | Topic | Decision | By | Accepted | Where |
| --- | --- | --- | --- | --- | --- |
| 2026-11-16 | Comparison of teehr and the core | Where teehr and teehr-core (VeriRace's metrics) differ on the same inputs, the core's definition is the shared one; teehr's bugs (Spearman's ddof, NSE's sum-is-zero guard) are fixed in teehr | TEEHR | | #8, teehr#873 |
| 2026-11-16 | `success_ratio` | hits / (hits + false_alarms); teehr's calculation, (hits + correct_negatives) / count, becomes `proportion_correct` (alias `accuracy`) | TEEHR | | #8 |
| 2026-11-16 | `flow_duration_curve_slope` | the log form (Sawicz et al. 2011) over exceedances [0.33, 0.66]; `log: false` gives the linear slope; Q(e) is the quantile at level 1 − e by the shared quantile definition (snapped), not a plotting position | TEEHR | | #8 |
| 2026-11-16 | Signatures with gaps | computed over complete pairs, the same sample as every other metric | TEEHR | | #8 |
| 2026-11-16 | Upload contract | `standards/uploads.md`: four steps, kinds and formats (Shapefile, GeoPackage), statuses with `awaiting_file` and cancel, the preview (`needed`, `notes`, `ingest`), all problems at once, mapping fields named for their column (wide files, `time_format`, explicit `timezone`, `location_key`, `left` / `right`), result fields | TEEHR | VeriRace | #6 |
| 2026-11-16 | Layer geometry exchange | WKB in EPSG:4326 in `geometry`: GeoArrow in Arrow, GeoParquet in Parquet, GeoJSON in JSON and as a FeatureCollection (`id` = `location_key`), `bbox` | TEEHR | VeriRace | #6 |
| 2026-11-16 | Error entries | 422 `errors` entries `{loc, msg, type}`, `loc` starting `body` / `query` / `path`; shared `code` `token_refresh_required` | TEEHR | VeriRace | #6 |
| 2026-11-16 | Bootstrap procedure | VeriRace's, pinned in `standards/bootstrap.md`: per-group seeds from the stage seed and group key, three streams, block draws per method, original sample excluded, NaN replicates dropped, type-7 quantiles; teehr adopts it in Phase 4 | TEEHR | VeriRace | #5 |
| 2026-11-16 | Bootstrap names | `year_block` canonical, `gumboot` its alias; `year_start_month` | TEEHR | VeriRace | #5 |
| 2026-11-16 | Seed key encoding | `group_by` order, compact JSON, timestamps ISO 8601 UTC with `Z`, whole floats with `.0`, `null` for missing | TEEHR (VeriRace: `null`, order) | both | #5 |
| 2026-11-16 | Sub-day blocks | A duration that isn't whole days becomes rows per group (`round(seconds × (n − 1) / span)`, halves to even, in `[1, n]`); not drawable from state | VeriRace | TEEHR | #5 |
| 2026-11-16 | `min_years` default | 5 | TEEHR | VeriRace | #5 |
| 2026-11-16 | Metric aliases | `variability_ratio` → `relative_standard_deviation` (VeriRace's CV ratio becomes `coefficient_of_variation_ratio`); `relative_mean` → `multiplicative_bias`; teehr's short names (`relbias`, `multbias`, `relmean`, `relstd`, `vr`, `kge_mod1`, `kge_mod2`) | TEEHR | VeriRace | #4 |
| 2026-11-16 | Contingency outputs | The four counts as metrics, not `confusion_matrix` | VeriRace | TEEHR | #4 |
| 2026-11-16 | `max_value_time_delta` | Seconds, as a number | TEEHR | VeriRace | #4 |
| 2026-11-16 | Year parameter | `year_start_month` (calendar years at 1, US water years at 10), for `annual_peak_flow_bias` (default 1) and `year_block`; `year_block` stays the method name, `gumboot` an alias | TEEHR | VeriRace | #4 |
| 2026-11-16 | KGE weights | `sr`, `sa`, `sb` (default 1); NaN when either std is 0, whatever the weights | TEEHR | VeriRace | #4 |
| 2026-11-16 | Transforms | `log` (natural), `sqrt`, `square`, `cube`, `exp`, `reciprocal` (alias `inv`), `abs`; undefined pairs left out; none on contingency, Brier, ensemble and rank metrics | VeriRace (list: TEEHR) | both | #4 |
| 2026-11-16 | `add_epsilon` | `1e-6` in the evaluation unit, to denominators and before `log` / `reciprocal` | VeriRace | TEEHR | #4 |
| 2026-11-16 | Result units | Evaluation unit, its square, seconds, or none, per metric | TEEHR | VeriRace | #4 |
| 2026-11-16 | Event rule, CRPS default | Events are values `≥` the threshold; CRPS estimator `standard` | VeriRace | TEEHR | #4 |
| 2026-11-16 | Metric names | One canonical name per metric (teehr's long form); frozen aliases (`nse`, `kge`, ...) accepted in specs; results labelled with the canonical name | TEEHR | VeriRace | #3, `standards/metrics.md` |
| 2026-11-16 | Metrics registry | `standards/metrics.md`, seeded from teehr's definitions, reconciled with VeriRace's | TEEHR | VeriRace | #3 |
| 2026-11-16 | Spec `version` | Starts at 1 | TEEHR | VeriRace | #3 |
| 2026-11-16 | Durations | ISO 8601 everywhere | TEEHR | VeriRace | #3 |
| 2026-11-16 | Bootstrap | On the aggregate stage: `method`, `block`, `reps`, `seed`; `iid`, `circular_block`, `stationary`. Open: `year_block` vs `gumboot`, how an interval is asked for | TEEHR | VeriRace | #3 |
| 2026-11-16 | Pair window columns | One `period_start` / `period_end` | TEEHR | VeriRace | #3 |
| 2026-11-16 | Quantile snapping | On every path, for metrics marked mergeable; others may be exact | VeriRace (TEEHR: mergeable only) | both | #3, `standards/metric-state.md` |
| 2026-11-16 | Sketch unit | Sketches are built in the evaluation unit; the zero floor is in it | TEEHR | VeriRace | #3 |
| 2026-11-16 | Quantile definition | Mean of the order-statistic representatives at `q·n`; no range collapsing; zero floor `1e-12` | VeriRace | TEEHR | #3 |
| 2026-11-16 | Metric state | Fixes the math, not the storage; state rows aren't exchanged; SI-keyed state keys dropped | VeriRace | TEEHR | #3 |
| 2026-11-16 | `log` | No bare `log`; `ln` and `log10` | VeriRace | TEEHR | #3, `spec/expressions.md` |
| 2026-11-16 | Allow-list additions | Safe candidates join, unsafe stay off, three to check | TEEHR | VeriRace | #3, `spec/expressions.md` |
| 2026-11-16 | Units in input filters (interim) | Numbers in an input `filter` are in the variable's canonical unit; pair-level ones in the evaluation unit | TEEHR | VeriRace | #3, `spec/expressions.md` |
| 2026-11-16 | Differences and normalizers | Convert by scale alone (a 5 K ramp is a 5 Cel ramp) | VeriRace | TEEHR | #3, `standards/units.md` |
| 2026-11-16 | `output.unit` | The evaluation unit; every `{value, unit}` parameter is converted to it | VeriRace | TEEHR | #3, `standards/units.md` |
| 2026-11-16 | Dataset references | `<name>` (the evaluation's project) or `public.<name>`; `<project>.<name>` may come later | TEEHR | VeriRace | #3, `spec/README.md` |
| 2026-11-16 | Per-series period | Both systems let a series override its dataset's period (one per series) | TEEHR (correction) | VeriRace | #3 |
| 2026-11-16 | Data API | `/data` is the shared target; each system's gaps listed as known differences | VeriRace | TEEHR | #3, `standards/data-api.md` |
| 2026-11-16 | Variables | Each system keeps its own registry with canonical units; specs don't depend on either | VeriRace | TEEHR | #3, `standards/units.md` |
| 2026-11-15 | Final say | TEEHR decides when the teams can't agree | TEEHR | VeriRace | #2, `GOVERNANCE.md` |
| 2026-11-13 | Explicit units | Every unit-bearing spec parameter is `{value, unit}`; bare numbers refused | TEEHR | VeriRace | shared contract v0, `standards/units.md` |
| 2026-11-13 | Shared variable list | Declined: variables aren't shared | VeriRace (proposed) | TEEHR declined | shared contract v0 |
