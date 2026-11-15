# Governance

## What belongs here

Anything a spec depends on to mean the same thing on both systems: the spec structure, the
expression language, unit and period rules, metric state, the shared column names, and the data
API contract. Not: storage layout, registries one system keeps for itself, or deployment.

When a system decides differently on something that isn't shared, note it under "Known
differences" in the relevant standard so nobody has to rediscover it.

## Making a change

1. Open a PR. Say what changes and why, and which conformance cases it adds or changes.
2. One maintainer from each team approves.
3. Merge, and add a `CHANGELOG.md` entry with the new `contract_version`.

Each system then adopts the change in its own repo and runs the conformance cases.

## Maintainers

| Team | Maintainers |
| --- | --- |
| TEEHR | @mgdenno |
| VeriRace | (to add) |

Add each team's maintainers here; the repo is private, so access is by invitation.
