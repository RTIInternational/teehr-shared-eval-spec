# teehr-shared-eval-spec

The evaluation spec and standards that TEEHR and VeriRace both implement, so a saved spec means
the same thing, and gives the same numbers, on either system.

This repo holds only what both systems must agree on. Each system's own choices (how it stores
data, its registries, its deployment) stay in that system's repo.

## Contents

| Path | What |
| --- | --- |
| `spec/` | the evaluation spec: structure, validation, versioning |
| `spec/expressions.md` | the `filter` / `expr` expression language |
| `standards/units.md` | units on spec parameters |
| `standards/periods.md`, `periods.csv` | temporal extent: period forms and the shared rows |
| `standards/metric-state.md` | mergeable metric state |
| `standards/timeseries.md` | the shared timeseries column names and types |
| `standards/data-api.md` | reading data over HTTP: paging, formats, errors |
| `conformance/` | cases both systems run in CI |
| `CHANGELOG.md` | every agreed change |

## Changes

A change is a pull request. It is agreed when one maintainer from each team approves it, and
merging it is the decision (see `GOVERNANCE.md`). Discussion happens on the PR, not by email.

## Versioning

`contract_version` covers everything here and follows semver. A change that alters a metric value
or rejects a previously valid spec is a major bump, with the reason in `CHANGELOG.md`.
