# Governance

## What belongs here

Anything a spec depends on to mean the same thing on both systems: the spec structure, the
expression language, unit and period rules, metric state, the shared column names, and the data
API contract. Not: storage layout, registries one system keeps for itself, or deployment.

When a system decides differently on something that isn't shared, note it under "Known
differences" in the relevant standard so nobody has to rediscover it.

## Making a change

1. Open a PR. Say what changes and why, and which conformance cases it adds or changes.
2. One maintainer from each team approves. If the teams can't agree, TEEHR decides (TEEHR
   ADR-0011): the shared contract is TEEHR's, and other systems adopt it.
3. Merge, and add a `CHANGELOG.md` entry with the new `contract_version`.
4. Record each decision in `DECISIONS.md`: who proposed it, who accepted it, and where.

## Who wrote what

Both teams (and their agents) work under one GitHub login, so GitHub's author fields can't tell
them apart. Say it in the content instead:

- PR titles, review summaries and commit subjects start with `[TEEHR]` or `[VeriRace]`.
- Commits carry a trailer: `Team: TEEHR` or `Team: VeriRace`.
- A PR's branch is changed by the team that opened it. The other team reviews; when the authors
  hand it over (or ask), the reviewer may commit to it, labelled as above.

Each system then adopts the change in its own repo and runs the conformance cases.

## Maintainers

| Team | Maintainers |
| --- | --- |
| TEEHR | @mgdenno |
| VeriRace | (to add) |

Add each team's maintainers here; the repo is private, so access is by invitation.
