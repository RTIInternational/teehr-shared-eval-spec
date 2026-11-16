# Expression language (v0)

- **Status:** agreed 2026-11-13 (shared contract v0, section B). Units in expressions are open.

Used by `filter` (row filters) and `fields[].expr` (derived columns). **Not raw SQL.**

- **Form:** a restricted SQL-expression subset, checked against an allow-list. No subqueries, no
  functions off the list, no DDL/DML.
- **Canonical dialect:** expressions are stored in an engine-neutral standard-SQL subset. Each
  engine transpiles with sqlglot to its own dialect (Trino/Spark for TEEHR, DuckDB for VeriRace).
- **Identifiers:** the paired-table columns `primary_value`, `secondary_value`, `value_time`,
  `reference_time`, `primary_series_key`, `secondary_series_key`, `secondary_dataset`, `member`,
  `quantile`, `period_start`, `period_end`, plus any `fields` already defined. An input's `filter`
  sees that dataset's own columns.
- **Allow-list v0:**
  - operators: `= != < <= > >=`, `AND OR NOT`, `+ - * /`, `IN`, `BETWEEN`, `IS [NOT] NULL`, `LIKE`
  - functions: `abs`, `coalesce`, `least`, `greatest`; time: `hour`, `day`, `month`, `year`,
    `date_trunc`; math: `ln`, `log10`, `exp`, `sqrt`, `pow` (or `power`); conditional: `CASE WHEN`
  - literals: numbers, quoted strings, ISO-8601 timestamps
- Off-list expressions are rejected at submit, before any run. The allow-list is versioned with
  the spec.
- **No bare `log`.** It means different things per engine: base 10 in DuckDB, natural log in
  Spark, and Trino has no one-argument form. sqlglot passes `log(x)` through unchanged, so the same
  expression would silently give different numbers. Use `ln` or `log10`.

## Candidates for the allow-list

VeriRace accepts these today: `ceil`, `floor`, `round`, `nullif`, `quarter`, `week`, `minute`,
`dayofyear`, `dayofweek`, `date_diff`, `epoch`, `extract`, `isnan`, `isinf`, `isfinite`, `lower`,
`upper`, `length`, `substring`, `concat`, plus `CAST`, `ILIKE`, `||` and `%`. Those that transpile
faithfully to Trino and Spark join the list; VeriRace drops the rest.

## Open: units in expressions

A number compared against a unit-bearing column (`value > 1000`) needs a unit
(`standards/units.md`), but there is no syntax for it yet. Agreed order: thresholds get units
first; expressions follow. Options to decide here:

- a unit-bearing literal in the grammar (e.g. `value > 1000 '[cft_i]/s'`), converted at validation;
- a function, e.g. `value > qty(1000, '[cft_i]/s')`;
- moving value bounds out of expressions into structured filter fields that carry `{value, unit}`.

Proposed interim (not yet agreed): until decided, a numeric literal in an input's `filter` is read
in that dataset's own unit, and one in a pair-level `filter` or `fields[].expr` is read in the
evaluation unit (`output.unit`). After pairing, both sides are in the evaluation unit, so "the
dataset's own unit" no longer identifies one unit.
