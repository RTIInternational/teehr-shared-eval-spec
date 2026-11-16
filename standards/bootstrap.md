# Bootstrap

- **Status:** agreed 2026-11-16 (proposed by TEEHR, accepted by VeriRace). The procedure is VeriRace's
  (`verirace/metrics/bootstrap.py`); teehr adopts it with its metric kernels (TEEHR Phase 4).

A spec's `bootstrap` (on an aggregate stage) gives each metric an interval by resampling each
group many times. Random draws make the results depend on details no spec mentions, so this pins
them: the same spec, data and seed give the same intervals on both systems.

## Settings

| Field | Meaning | Default |
| --- | --- | --- |
| `method` | `iid`, `circular_block`, `stationary`, `year_block` | `circular_block` |
| `block` | block length for `circular_block` / `stationary`: whole days or months as ISO 8601 (`P7D`, `P1M`); another duration (`PT12H`), converted to rows per group ("Units"); or a number of rows | required for those two; none for `iid`, `year_block` |
| `reps` | replicates, 10 to 10,000 | 1000 |
| `seed` | the stage seed, ≥ 0 | 0 |
| `quantiles` | the interval: quantiles of the replicates, each in (0, 1) | `[0.05, 0.95]` |
| `sort_by` | pair columns ordering each group before blocks are drawn | `[value_time]` |
| `min_count` | fewer forecast cases with values than this: null interval | 30 |
| `year_start_month` | `year_block`: the month a year starts in (10 = US water year, 1 = calendar) | 10 |
| `min_years` | `year_block`: fewer distinct years than this: null interval | 5 |

**Alias:** `gumboot` is `year_block` (teehr's name; its default year starts in October, as here).

## Units

A replicate resamples **units** of a group, in order:

- **Cases.** A unit is a forecast case: the members of one ensemble forecast are resampled together.
  Deterministic pairs are one case each.
- **`year_block`:** a unit is a year. A row's year is its calendar year, plus one when
  `year_start_month` > 1 and its month is ≥ `year_start_month`. Each replicate draws as many whole
  years as the group spans, with replacement (`iid` over years), so each year keeps its seasonality.
- **Day or month blocks** (`block` in whole days or months): a unit is one day or month of
  `value_time`, counted from the group's first to its last, empty ones included, so a block spans
  the same time however dense the pairs. This is what lets a bootstrap be drawn from stored state
  (`standards/metric-state.md`) and give the same intervals as one drawn from pairs.

- **Other durations** (`block` such as `PT12H`): units are cases, and the block is a number of
  rows for each group: `round(seconds × (n − 1) / span)`, rounding halves to even, clipped to
  `[1, n]`, where `n` is the group's cases and `span` its first-to-last `value_time` in seconds (a
  block of 1 when `span` is 0). These are rows, not periods, so they can't be drawn from state.

Cases and years are numbered `0 .. n−1` in order of first appearance after sorting by `sort_by`.
A day or month is numbered by its offset from the group's first day or month, empty ones counted.

## Draws

Each group has its own random streams, so results don't depend on how groups are split across
workers or how replicates are batched.

1. **Group seed:** `blake2b` (8-byte digest) of the JSON array `[seed, key_1, key_2, ...]`, read
   as a little-endian unsigned integer:
   - the key values in the stage's `group_by` order;
   - compact separators (`,` and `:`), no spaces;
   - a timestamp as an ISO 8601 UTC string with `Z` (`"2024-01-01T00:00:00Z"`), with fractional
     seconds only when non-zero;
   - a float that's a whole number with `.0` (`1.0`); a missing value as `null`.
2. **Streams:** `SeedSequence(group_seed).spawn(3)` gives `starts`, `breaks` and `more`, each a
   PCG64 generator (numpy `default_rng`).
3. **Blocks** over `n` units, for `reps` replicates:
   - `iid` and `year_block`: blocks of 1; `k = n` per replicate.
   - `circular_block`: blocks of `b = min(block, n)`; `k = ⌈n / b⌉` per replicate, the last cut to
     fill `n`.
   - Both: `starts.integers(0, n, size=(reps, k))`, drawn as one array, replicate by replicate.
   - `stationary` (Politis & Romano 1994): with `b = min(block, n)`, `p = 1 / b`,
     `mean = n / b` and `m = min(n, ⌈mean + 6·√mean + 6⌉)` draws per replicate: `lengths = breaks.geometric(p, size=(reps, m))`, then
     `starts = starts.integers(0, n, size=(reps, m))`. Blocks are used in order until they cover `n`
     (the last cut to fit). A replicate they don't cover draws more, one block at a time, from
     `more`: a length `more.geometric(p)`, then a start `more.integers(0, n)`.
   - Blocks wrap around the end of the series.
4. **Weights:** a replicate's weight for a unit is how many times its blocks cover it. A metric of a
   replicate is the metric of the pairs weighted that way (the same as computing it from the
   resampled pairs).

## Intervals

- The original sample is **not** a replicate.
- Replicates where a metric is undefined (NaN) are left out; with none left, the interval is null.
- Quantiles of the remaining replicates, linear interpolation (Hyndman & Fan type 7, numpy's
  default).
- A group below `min_count` cases with values, or (`year_block`) below `min_years` years, gets a
  null interval.

## Known differences (until teehr adopts this)

| | This standard | teehr today |
| --- | --- | --- |
| Seed | per group, from the stage seed and the group key | one seed for every group (arch's generator), so groups of the same size get the same draws |
| Original sample | not a replicate | `gumboot` counts it as replicate 0 |
| Undefined replicates | left out | make the interval NaN |
| Null intervals | `min_count`, `min_years` | minimum sample size, mean and variance checks |
| Block draws | as above | arch's `CircularBlockBootstrap` / `StationaryBootstrap` |

## Decided

Recorded in `DECISIONS.md` (2026-11-16): this procedure; `year_block` the canonical name with
`gumboot` its alias; `year_start_month`; the key encoding above (VeriRace changes from Python's
`default=str` timestamps, so its seeds and intervals change once); `min_years` default 5;
durations that aren't whole days converted to rows per group.
